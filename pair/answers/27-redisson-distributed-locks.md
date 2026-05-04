# A31 — Redisson Distributed Locks: Deep Dive

## 1. Why `synchronized` fails in a multi-pod deployment

`synchronized`, `ReentrantLock`, `StampedLock` — all the `java.util.concurrent` locks — are **in-process monitors backed by JVM object headers / AQS state**. The lock state lives in the heap of one JVM.

In Kubernetes with, say, three replicas of the booking service behind a Service:

1. User Alice's request is routed by kube-proxy to pod A.
2. User Bob's request for the same seat is routed to pod B.
3. Pod A's `synchronized(seat)` takes the lock *on its own heap*.
4. Pod B's `synchronized(seat)` takes a completely independent lock on *its* heap — it knows nothing about A.
5. Both pods see the seat as free, both write `booked=true`, the second `INSERT` into `booking` either overwrites or races the first. Classic double-book.

The JVM's memory model cannot reach across processes, let alone across nodes. You need an external coordination point — Redis, Zookeeper, a DB row lock, etc. A `ReentrantLock` inside one pod is at best useless here, at worst misleading because local testing looks fine.

## 2. What Redisson does at the Redis command level

`RLock` is implemented as a **Lua-scripted hash entry** with a TTL. When `tryLock` succeeds, Redisson executes a Lua script roughly equivalent to:

```lua
if (redis.call('EXISTS', KEYS[1]) == 0) then
    redis.call('HSET',   KEYS[1], ARGV[2], 1)    -- ARGV[2] = uuid:threadId
    redis.call('PEXPIRE', KEYS[1], ARGV[1])      -- ARGV[1] = leaseMillis
    return nil                                    -- acquired
end
if (redis.call('HEXISTS', KEYS[1], ARGV[2]) == 1) then
    redis.call('HINCRBY',  KEYS[1], ARGV[2], 1)   -- reentrant
    redis.call('PEXPIRE',  KEYS[1], ARGV[1])
    return nil
end
return redis.call('PTTL', KEYS[1])                -- ms until free
```

Key points:
- The lock is a **hash** whose field is `<redissonUUID>:<threadId>`. This encodes *owner*, enabling reentrancy and safe unlock.
- **Reentrant**: same owner → `HINCRBY` bumps a counter; `unlock` decrements; the key is `DEL`'d only when the counter reaches zero.
- **Release is ownership-checked in Lua** — a compare-and-del; you cannot accidentally release someone else's lock after your lease expired.
- **If not acquired**, the script returns the remaining TTL. The client subscribes to a per-lock pub/sub channel (`redisson_lock__channel:<name>`) and sleeps until either the TTL elapses or the current holder publishes "released". This is how contended waits avoid busy-polling.

## 3. `RLock` vs `RFairLock` vs `RReadWriteLock`

- **`RLock`** — standard mutex, reentrant, **not** FIFO. Whoever happens to win the pub/sub wakeup race acquires the lock next. High throughput, but a waiter can be starved under continuous contention.
- **`RFairLock`** — maintains a Redis **list/queue** of waiters plus a per-waiter timeout. Acquisition order matches request order. Cost: more Redis round-trips per acquire/release, and the queue itself is a failure domain (needs its own TTL cleanup). Use it when ordering matters — e.g. ticket queues where "first in line" is a user-facing promise, or workflows with bounded per-client fairness SLAs.
- **`RReadWriteLock`** — implements `java.util.concurrent.locks.ReadWriteLock` semantics over Redis. Multiple concurrent readers share the lock; writers are exclusive and drain readers. Use when reads dominate and you need consistency with occasional writers — e.g. a cached configuration snapshot that is updated once an hour, but read on every request.

In a seat-booking flow you want `RLock`: fairness is irrelevant (the seat is either taken or not), and the critical section is short.

## 4. The watchdog

When you call `tryLock()` **without an explicit `leaseTime`** (or pass `-1`), Redisson engages the watchdog:

- Initial lease = `Config.lockWatchdogTimeout`, default **30 seconds**.
- A scheduled task on the Redisson client **renews** the lease every `watchdogTimeout / 3` (so every 10s by default) by re-issuing `PEXPIRE`.
- Renewal continues as long as the JVM holds the lock. On `unlock()`, the renewal task is cancelled. On JVM crash, renewal stops and the lock self-expires after at most 30s.

This makes locks "crash-safe but long-running-friendly": a business operation that legitimately takes 45s won't have its lock stolen, but a crashed pod doesn't leave a lock pinned forever.

**When to disable the watchdog by passing an explicit `leaseTime`:**

```java
lock.tryLock(2, 10, TimeUnit.SECONDS);   // wait up to 2s, then lease for exactly 10s
```

- You need a **hard upper bound** on how long any single pod can hold the lock — typically for SLA reasons or to cap the blast radius of a stuck thread.
- You're running short critical sections and want to avoid the scheduled renewal task's overhead (~1 Redis round-trip every 10s per held lock).
- You run **many** concurrent locks per pod and renewals would add up to non-trivial Redis load.
- Your operation is idempotent enough that a lease expiry mid-work is safe.

Rule of thumb: let the watchdog handle it unless you have a specific reason. Pick explicit leases for short, idempotent critical sections; keep the watchdog for long orchestrations where you can't predict duration.

## 5. Redlock: Kleppmann vs. antirez

**Redlock** (antirez, 2014) is an algorithm for locking across N independent Redis masters (not a cluster). A client acquires the lock on a majority (N/2+1) of nodes within a validity window, then validates the elapsed time is less than the lease — if so, it holds the lock for `leaseTime - elapsed`. Releases go to all N.

**Kleppmann's 2016 critique** ("How to do distributed locking"):

1. **Clock drift**: Redlock's safety relies on clocks that don't jump. A sysadmin stepping the wall clock, an NTP slew, a VM live-migration pause, or a long GC pause can cause one client to overshoot its lease while still believing it holds the lock. Concurrent holders, data corruption.
2. **Fencing tokens**: No monotonically increasing token is issued. A storage backend can't detect that a "stale" holder is writing after its lock expired. Properly-safe distributed locks (Chubby, Zookeeper) issue fencing tokens so the downstream resource can reject out-of-order writes.
3. **Process pauses**: stop-the-world GC or OS-level pauses can effectively freeze a client past its lease; the lock is re-acquired elsewhere; when the paused client wakes it happily proceeds thinking it still owns the lock.

**antirez's 2016 reply**:

1. Redlock assumes bounded clock drift; operators can enforce NTP discipline. Not great, but tractable in ops.
2. Fencing tokens are a separate, orthogonal concern — nothing stops you from layering one on top (and indeed Redisson can combine a lock with a monotonic counter).
3. GC pauses affect all locking systems, not only Redlock. A properly-tuned JVM + heartbeat-based mitigation works.

Kleppmann maintained that if your correctness depends on mutual exclusion, use a system that provides linearizability (Zookeeper, etcd/Raft, Consul). For best-effort optimization use cases, Redis locks are fine.

**What this means for seat booking**:

- The **authoritative truth** is your DB row or uniqueness constraint (`UNIQUE(show_id, seat_no)`), not the lock. The lock is an *optimization* that prevents spurious retries and reduces DB contention.
- Even if Redlock gets it wrong once in a blue moon, the `UNIQUE` constraint turns a double-book into a failed insert on the loser — user sees "seat just taken, try another", nobody gets two different seats.
- Do **not** rely on the lock as the sole source of mutual exclusion. Always combine with a DB-level idempotency or uniqueness guarantee. That is the real answer to Kleppmann: cheap Redis locks for the hot path, DB invariants for correctness.

## 6. The subtle correctness bug in `book()` and the fix

Two kinds of failures can duplicate a booking even with the lock held correctly:

- **The client retries the HTTP call** (e.g. network blip, client-side timeout before the 200 OK came back). First call succeeded and the lock was released normally; second call takes the lock again, sees `s.isBooked() == false` is **not** the problem — the check would catch it — but sees `isBooked == true` and throws `AlreadyBookedException`, even though *this same user* already holds that booking. The user gets told "seat taken" when in fact they took it. Worse, if the retry happens *before* the first DB commit is visible (replica lag, read-your-writes failure), both requests see the seat as free, both hold the lock serially, both insert → double booking.

- **Redlock edge case** (clock drift / GC pause) → two holders think they hold the lock simultaneously. Exactly Kleppmann's scenario.

**Fix: idempotency key as the ultimate backstop.**

```java
public Booking book(String showId, String seatNo, String userId, String idemKey) {
    RLock lock = redisson.getLock("seat:" + showId + ":" + seatNo);
    try {
        if (!lock.tryLock(2, 10, TimeUnit.SECONDS)) throw new BusyException();

        // 1. Idempotency check FIRST — same key → return prior result.
        Optional<Booking> prior = bookingRepo.findByIdemKey(idemKey);
        if (prior.isPresent()) return prior.get();

        Seat s = seatRepo.find(showId, seatNo);
        if (s.isBooked()) throw new AlreadyBookedException();
        s.markBooked(userId);

        // 2. DB uniqueness as the hard invariant.
        try {
            return bookingRepo.save(Booking.of(showId, seatNo, userId, idemKey));
        } catch (DataIntegrityViolationException e) {
            // UNIQUE(show_id, seat_no) caught it — lock must have been wrong.
            return bookingRepo.findByShowAndSeat(showId, seatNo).orElseThrow();
        }
    } finally {
        if (lock.isHeldByCurrentThread()) lock.unlock();
    }
}
```

Schema requirements:
- `UNIQUE(show_id, seat_no)` on `seat` or `booking` — *real* mutual exclusion.
- `UNIQUE(idem_key)` on `booking` — retry-safe.

The lock stays for performance; the DB constraints carry correctness. This is the pattern Kleppmann would sign off on.

## Interview-ready answer

> "Redisson's `RLock` is a Lua-scripted hash keyed on `<uuid>:<threadId>` with a TTL — that encoding gives you ownership-checked unlock and reentrancy. Waiters subscribe to a pub/sub channel to avoid busy-polling. `RFairLock` adds a FIFO queue, and `RReadWriteLock` gives you reader/writer semantics. The watchdog is Redisson's crash-safety story: without an explicit leaseTime it defaults to 30s and renews every 10s; disable it by passing an explicit lease when you need a hard cap on hold time or want to avoid renewal traffic. On the Kleppmann debate — Redis locks are not linearizable, so I never use them as the single source of truth. For seat booking, the DB `UNIQUE(show, seat)` constraint is the correctness backstop; the lock is an optimization to reduce contention. An idempotency key on the booking row handles client retries and Redlock edge cases alike."
