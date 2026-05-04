# Q31 — Redisson Distributed Locks: Deep Dive

Existing Q21 covered the big picture (counters, Lua, holds, outbox). This one zooms into the locking primitives themselves.

```java
@Service
public class SeatBookingService {
    private final RedissonClient redisson;
    private final SeatRepo seatRepo;
    private final BookingRepo bookingRepo;

    public Booking book(String showId, String seatNo, String userId, String idemKey) {
        RLock lock = redisson.getLock("seat:" + showId + ":" + seatNo);
        try {
            if (!lock.tryLock(2, 10, TimeUnit.SECONDS)) {
                throw new BusyException("seat contended");
            }
            // critical section
            Seat s = seatRepo.find(showId, seatNo);
            if (s.isBooked()) throw new AlreadyBookedException();
            s.markBooked(userId);
            return bookingRepo.save(Booking.of(showId, seatNo, userId, idemKey));
        } finally {
            if (lock.isHeldByCurrentThread()) lock.unlock();
        }
    }
}
```

## Questions

1. Why does `synchronized` (or a `ReentrantLock`) fail to protect `book()` once the service is deployed to a multi-pod K8s setup? What exactly goes wrong at runtime?
2. How does `RLock` implement a mutex across JVMs? Walk through what actually happens in Redis when `tryLock` succeeds — at the command level.
3. Contrast `RLock`, `RFairLock`, and `RReadWriteLock`. When does the fair variant matter in practice?
4. Explain the Redisson *watchdog*. What is the default lease, when does it renew, and when should you pass an explicit `leaseTime` to disable it?
5. Summarise the Kleppmann vs. antirez Redlock debate: the specific failure scenarios Kleppmann raised, and what antirez countered. What does this mean for a seat-booking use case?
6. The `book()` method above has a subtle correctness bug even with the lock held correctly. Find it and fix it using the `idemKey`.
