# Q75 — Handling a Traffic Spike the System Cannot Absorb

Note: "Just autoscale" is the surface answer. The interviewer wants to hear about strategies that work *before*, *during*, and *when autoscaling isn't fast enough*.

## Questions

1. What is rate limiting? Name two algorithms used to implement it (token bucket, leaky bucket, fixed window, sliding window). How does Redis help here?
2. What is load shedding? How does it differ from rate limiting? When should a service drop requests on purpose?
3. What is backpressure? How do reactive systems (Spring WebFlux / Reactor) propagate it upstream?
4. What is a queue-based leveling pattern? Draw the flow and explain how it decouples producers from consumers during a spike.
5. Autoscaling solves sustained load but has warm-up latency. What do you do in the gap (typically 1–3 minutes) before new instances are ready?
6. What is a bulkhead? How does isolating thread pools or connection pools prevent one overloaded endpoint from taking down the whole service?
7. Describe circuit breaker's role in a traffic spike — does it help the overloaded service or its callers?
