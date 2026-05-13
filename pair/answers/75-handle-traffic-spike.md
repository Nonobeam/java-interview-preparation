# 75. Handling a Traffic Spike the System Cannot Absorb

Autoscaling is the correct answer but it has a 1–3 minute warm-up lag. The real interview question is: **what do you do before new instances are ready, and what prevents the existing instances from falling over?**

## Strategy map

```
Before the spike   →  During the gap (autoscale warming)  →  Sustained load
─────────────────     ──────────────────────────────────     ──────────────
Capacity planning     Rate limiting                           Autoscale
Load testing          Load shedding                          Queue-based leveling
CDN / caching         Circuit breaker                        Horizontal scaling
Queue buffering        Bulkhead isolation
```

## 1. Rate limiting

Controls how many requests a client or endpoint can make per time window.

**Algorithms:**
- **Token bucket**: a bucket refills at a fixed rate (e.g., 100 tokens/sec). Each request consumes one token. Allows short bursts.
- **Sliding window counter**: tracks request count in the last N seconds, no burst allowance.
- **Fixed window**: resets at the clock boundary — vulnerable to a "double burst" at the window edge.
- **Leaky bucket**: requests enter a queue and drain at a fixed rate. Smooths bursts.

**Redis implementation** (token bucket with `INCR` + TTL, or Lua script for atomicity):
```
INCR  user:123:req_count
EXPIRE user:123:req_count 60     -- reset every minute
if count > 1000: return 429 Too Many Requests
```

Rate limiting protects the system from a single bad actor or runaway client. It does not help if the spike is legitimate traffic from millions of users.

## 2. Load shedding

When the system is at capacity, **deliberately drop low-priority requests** rather than letting all requests degrade.

- Return `503 Service Unavailable` immediately when queue depth or CPU exceeds a threshold.
- Prioritise: health checks > paying users > free tier > crawlers.
- Better to serve 80 % of requests at normal latency than 100 % at 10x latency (or crashing).

Key difference from rate limiting: **rate limiting** controls per-client throughput; **load shedding** is a global valve the server opens when it can't keep up with anyone.

## 3. Backpressure (reactive systems)

Backpressure = upstream slows down when downstream is overwhelmed. This propagates the signal up the call chain rather than letting queues grow unboundedly.

In Spring WebFlux / Reactor:
- `Flux.onBackpressureDrop()` — drop elements when the subscriber can't keep up.
- `Flux.onBackpressureBuffer(maxSize)` — buffer up to N items, then error.
- `Flux.onBackpressureLatest()` — keep only the latest item.

In non-reactive systems, backpressure is implemented via bounded queues: if the queue is full, the producer blocks or rejects.

## 4. Queue-based leveling (traffic buffer)

Decouple request acceptance from request processing using a message queue:

```
Clients ──→ [HTTP Facade] ──→ [Queue (Kafka/SQS)] ──→ [Workers]
              (accepts fast,                           (process at
               returns 202 Accepted)                   their own pace)
```

- The facade accepts all requests instantly and queues them.
- Workers process at their own rate — if overloaded, the queue depth grows but nothing crashes.
- Callers get a job ID and poll for the result.

Ideal for: async operations (report generation, payment processing, email sending). Not suitable for: real-time, low-latency reads.

## 5. The autoscale warm-up gap

Autoscaling triggers after a metric breach (CPU > 70% for 2 minutes). New instances take 1–3 minutes to start and warm up (JVM JIT, Spring context loading, cache warming).

**Gap strategies:**
- Pre-warm: schedule autoscale before known traffic events (marketing campaigns, cron-triggered batch jobs).
- Keep minimum instances higher during peak hours (scheduled scaling policy).
- Cache aggressively (CDN, application cache) so fewer requests hit the origin during the gap.
- Load shedding absorbs the extra load while new instances spin up.

## 6. Bulkhead pattern

Isolate resources so one overloaded component cannot consume all shared resources.

```
┌──────────────────────────────────────────────────────┐
│ Thread pool: 200 threads total                       │
│  ┌─────────────────┐  ┌──────────────────────────┐   │
│  │ /checkout: 80   │  │ /search: 80              │   │
│  │ threads         │  │ threads                  │   │
│  └─────────────────┘  └──────────────────────────┘   │
│  ┌──────────────────────────────┐                    │
│  │ /health, /metrics: 40 threads│                    │
│  └──────────────────────────────┘                    │
└──────────────────────────────────────────────────────┘
```

If `/checkout` is flooded, it can only consume 80 threads — `/search` still works. Without bulkheads, one slow endpoint starves all others.

In Spring Boot with Resilience4j: `@Bulkhead(name = "checkout", type = THREADPOOL)`.

## 7. Circuit breaker during a spike

The circuit breaker helps the **callers** of an overloaded service, not the overloaded service itself.

- When a downstream service starts timing out/failing, the circuit breaker opens: all calls fail immediately without hitting the downstream service.
- This prevents cascade failures — your service's thread pool doesn't fill up waiting for a dead dependency.
- After a cooldown, the circuit tries a test request (half-open state). If it succeeds, the circuit closes.

## Interview one-liner

> "Autoscaling solves sustained load but has a warm-up gap. To survive the gap: rate limiting caps per-client throughput, load shedding drops requests when capacity is exhausted, and queue-based leveling decouples acceptance from processing. Bulkheads isolate thread pools so one overloaded endpoint doesn't take down the service, and the circuit breaker stops cascade failures from propagating to callers. These strategies work together — rate limiting + load shedding absorb the spike, queues smooth the curve, and autoscaling handles the sustained load once warm."
