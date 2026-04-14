## A21: Using Redis for Booking Concurrency

### Why Redis Beats DB Locking for Hot Counters

| Metric | DB row lock | Redis atomic op |
|--------|-------------|----------------|
| Latency | 5-20ms (network + disk + lock wait) | 0.1-1ms (memory, one hop) |
| Throughput | ~1,000 ops/sec per row | ~100,000 ops/sec per key |
| Contention behavior | Blocks or retries | Just decrements — no wait |
| Persistence | Guaranteed | Configurable (AOF / RDB) |

The DB is optimized for durability and consistency. Redis is optimized for speed. For a hot counter (last 10 containers on a popular voyage), a row lock becomes a bottleneck. Redis operations are **single-threaded and atomic** — there's no "race" to worry about.

---

### 1. Simple Atomic Counter — INCR / DECR

Redis commands like `INCR`, `DECR`, `INCRBY`, `DECRBY` are **atomic by design** — the Redis server processes them one at a time on a single thread.

```java
@Service
public class CapacityService {
    private final StringRedisTemplate redis;

    public boolean tryReserve(String voyageId) {
        String key = "capacity:voyage:" + voyageId;
        Long remaining = redis.opsForValue().decrement(key);
        if (remaining == null || remaining < 0) {
            redis.opsForValue().increment(key); // rollback
            return false;
        }
        return true;
    }

    public void release(String voyageId) {
        redis.opsForValue().increment("capacity:voyage:" + voyageId);
    }

    public void initCapacity(String voyageId, int capacity) {
        redis.opsForValue().set("capacity:voyage:" + voyageId, String.valueOf(capacity));
    }
}
```

**Problem with this code:** There's a small race window between the `DECR` returning -1 and the rollback `INCR`. Between those two calls, another client could see the wrong value. For most cases this is fine (self-healing), but for strict correctness use a Lua script.

---

### 2. Lua Scripts — Atomic Compound Logic

Lua scripts execute **atomically on the Redis server**. No other command runs during script execution. Use this when you need check-and-decrement as a single operation:

```lua
-- reserve.lua
local capacity = tonumber(redis.call('GET', KEYS[1]))
if capacity and capacity > 0 then
    redis.call('DECR', KEYS[1])
    return 1  -- success
else
    return 0  -- no capacity
end
```

Spring Boot usage:

```java
@Component
public class CapacityService {
    private final StringRedisTemplate redis;
    private final DefaultRedisScript<Long> reserveScript;

    public CapacityService(StringRedisTemplate redis) {
        this.redis = redis;
        this.reserveScript = new DefaultRedisScript<>();
        this.reserveScript.setLocation(new ClassPathResource("scripts/reserve.lua"));
        this.reserveScript.setResultType(Long.class);
    }

    public boolean tryReserve(String voyageId) {
        Long result = redis.execute(reserveScript,
            Collections.singletonList("capacity:voyage:" + voyageId));
        return result != null && result == 1L;
    }
}
```

**Benefits of Lua:**
- Atomic compound logic (no partial updates)
- Network round-trip reduced to one
- Server-side — no client logic to get wrong

---

### 3. Reservation Hold with TTL — Pending Bookings

Real bookings have two phases: reserve capacity, then confirm (after payment or operation checks). Holds must auto-expire if abandoned:

```lua
-- hold.lua
local capacity = tonumber(redis.call('GET', KEYS[1]))
if capacity and capacity > 0 then
    redis.call('DECR', KEYS[1])
    -- create hold record with TTL
    redis.call('SET', KEYS[2], ARGV[1], 'EX', ARGV[2])
    return 1
else
    return 0
end
```

```java
public boolean hold(String voyageId, String bookingId, int ttlSeconds) {
    List<String> keys = Arrays.asList(
        "capacity:voyage:" + voyageId,
        "hold:booking:" + bookingId
    );
    Long result = redis.execute(holdScript, keys, voyageId, String.valueOf(ttlSeconds));
    return result != null && result == 1L;
}
```

When the hold key expires, you restore capacity via a **Redis keyspace notification listener**:

```java
@Component
public class HoldExpiryListener implements MessageListener {

    @Override
    public void onMessage(Message message, byte[] pattern) {
        String key = message.toString(); // e.g. "hold:booking:B123"
        if (key.startsWith("hold:booking:")) {
            String bookingId = key.substring("hold:booking:".length());
            Booking b = bookingRepo.findById(bookingId).orElseThrow();
            if (b.getStatus() == BookingStatus.PENDING) {
                b.setStatus(BookingStatus.EXPIRED);
                capacityService.release(b.getVoyageId());
                bookingRepo.save(b);
            }
        }
    }
}
```

Configure keyspace notifications in redis.conf: `notify-keyspace-events Ex`.

---

### 4. Distributed Locks — SETNX / Redlock

For cases where you need a mutex across multiple services (e.g., only one worker can process a shipment at a time), use Redis as a distributed lock:

**Simple version — SET with NX + EX:**

```java
public boolean acquireLock(String resource, String token, long ttlMs) {
    Boolean acquired = redis.opsForValue()
        .setIfAbsent("lock:" + resource, token, Duration.ofMillis(ttlMs));
    return Boolean.TRUE.equals(acquired);
}

public void releaseLock(String resource, String token) {
    // Use Lua to prevent releasing a lock owned by someone else
    String script =
        "if redis.call('GET', KEYS[1]) == ARGV[1] then " +
        "  return redis.call('DEL', KEYS[1]) " +
        "else return 0 end";
    redis.execute(new DefaultRedisScript<>(script, Long.class),
        Collections.singletonList("lock:" + resource), token);
}
```

**Important:**
- `SETNX` (set if not exists) is atomic — guaranteed mutex
- TTL is mandatory — prevents deadlock if holder crashes
- Release must check ownership (Lua compare-and-delete) — else you might release another holder's lock after yours expired

**Redlock (for multi-node Redis):** If you run a Redis cluster, a single-node SETNX can fail during failover. Redlock acquires locks on majority of N nodes. Use Redisson library:

```java
RLock lock = redissonClient.getLock("booking:" + bookingId);
if (lock.tryLock(5, 30, TimeUnit.SECONDS)) {
    try {
        // critical section
    } finally {
        lock.unlock();
    }
}
```

**Caveat:** Martin Kleppmann famously criticized Redlock as not safe for correctness-critical use cases. For billing or regulatory guarantees, use a proper consensus system (Zookeeper, etcd). For booking holds, Redlock is fine.

---

### 5. Keeping Redis and DB Consistent — Outbox Pattern

Redis is fast but not durable in the same way as a DB. You MUST reconcile. Two main strategies:

**Strategy A: DB is source of truth, Redis is cache**
- On app startup, load capacity from DB into Redis
- Periodically rebuild Redis from DB (reconciliation job)
- If Redis dies, capacity recovers from DB
- Simpler, but has a reconciliation lag window

**Strategy B: Redis is hot path, DB via Outbox**
- Reserve in Redis (authoritative for hot counter)
- Write booking to DB transactionally with an `outbox_event` record
- A relay process reads outbox → publishes to Kafka → downstream services react
- DB rebuild of Redis only happens on startup or disaster recovery

```java
@Transactional
public Booking confirm(String bookingId) {
    // 1. Redis already decremented during hold phase — no DB contention
    Booking b = bookingRepo.findById(bookingId).orElseThrow();
    b.setStatus(BookingStatus.CONFIRMED);
    bookingRepo.save(b);

    // 2. Outbox event — same transaction as booking update
    outboxRepo.save(OutboxEvent.of("BookingConfirmed", b));
    return b;
}
```

A separate worker (or Debezium CDC) reads `outbox_event` and publishes to Kafka. This gives **exactly-once semantics** between the DB and message bus.

---

### 6. Capacity Splitting / Sharding

For extreme contention (airline last-seat syndrome), split one counter into N sub-counters:

```
capacity:voyage:V1:shard:0 = 250
capacity:voyage:V1:shard:1 = 250
capacity:voyage:V1:shard:2 = 250
capacity:voyage:V1:shard:3 = 250
```

Client randomly picks a shard. If it's empty, fall back to another shard. Total capacity = sum of shards. Scales linearly with shard count.

**Use case:** Peak-season booking for major trade lanes.

---

### 7. When NOT to Use Redis

- **Low-volume internal tools** — over-engineering. Use `@Version` or atomic SQL.
- **Strict financial correctness** — Redis is not ACID by default. Use a proper DB transaction.
- **You need transactional joins** — Redis doesn't do joins or complex queries.
- **You can't operate Redis** — introduces ops burden (monitoring, replication, failover).

---

### Summary: Redis Booking Patterns

| Pattern | Use case |
|---------|----------|
| `DECR` / `INCR` atomic counter | Simple capacity tracking |
| Lua script | Compound check-and-decrement atomically |
| `SET key value EX ttl` | Reservation hold with auto-expiry |
| Keyspace notifications | React when a hold expires |
| `SETNX` / Redlock | Distributed locks across services |
| Outbox pattern | Keep DB and Redis consistent |
| Capacity sharding | Extreme contention, split hot key into N |

---

### Interview-Ready Answer

> "The Redis approach pushes contention out of the database. For a booking capacity counter, a DB row lock gives maybe 1,000 ops/sec and every client has to wait on the lock. Redis can do `DECR` at 100,000 ops/sec with no lock wait because every command is atomic and single-threaded.
>
> Basic pattern: `DECR capacity:voyage:V1`. If result is negative, rollback with `INCR`. For compound logic, put it in a Lua script — Redis runs the whole script atomically.
>
> For reservation holds, I use `SET hold:booking:B123 {data} EX 300` — Redis auto-expires the key after 5 minutes. A keyspace notification listener restores capacity when the hold expires.
>
> The tricky part is keeping Redis and the DB consistent. I use the outbox pattern — the DB is updated transactionally with an `outbox_event` record, and a relay process publishes events to Kafka. Debezium can automate this via CDC.
>
> For distributed locks, `SET NX EX` is enough for most cases. Redlock or Redisson adds multi-node safety, though for strict correctness you want a proper consensus system like Zookeeper — Redlock has known edge cases during network partitions."
