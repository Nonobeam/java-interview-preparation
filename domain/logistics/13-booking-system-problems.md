## D13: Common problems when building a booking system

---

### 1. Double Booking / Race Condition

**Problem:** Two users book the last available slot at the same time. Both pass the availability check, both get confirmed → over-allocated.

**Solution: Pessimistic locking**
```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT s FROM Slot s WHERE s.id = :id")
Slot findByIdForUpdate(@Param("id") Long id);
```
Or **optimistic locking** with version field:
```java
@Version
private Long version; // throws OptimisticLockException on conflict
```
Or reduce to a **single atomic decrement** in DB:
```sql
UPDATE slot SET available = available - 1
WHERE id = ? AND available > 0
```

---

### 2. Overbooking / Capacity Management

**Problem:** How do you track remaining capacity across containers, vessels, or time slots in real time?

**Solution:** Maintain a `capacity` table and update atomically. Use Redis for high-throughput counters:
```java
// Redis atomic decrement — returns remaining capacity
Long remaining = redisTemplate.opsForValue().decrement("capacity:voyage:0123E");
if (remaining < 0) {
    redisTemplate.opsForValue().increment("capacity:voyage:0123E"); // rollback
    throw new NoSpaceAvailableException();
}
```
For less critical paths, a DB-level check-and-decrement in a single transaction is sufficient.

---

### 3. Reservation Expiry (Hold & Release)

**Problem:** User starts a booking, holds a slot, then abandons it. Slot is locked forever.

**Solution:** Two-phase booking with TTL:
1. **PENDING** — slot reserved for N minutes
2. **CONFIRMED** — payment/action completed
3. **EXPIRED** — auto-released after TTL

```java
// Scheduled job to release expired holds
@Scheduled(fixedDelay = 60_000)
public void releaseExpiredBookings() {
    List<Booking> expired = bookingRepo.findByStatusAndExpiredAtBefore(
        BookingStatus.PENDING, LocalDateTime.now()
    );
    expired.forEach(b -> {
        b.setStatus(BookingStatus.EXPIRED);
        capacityService.release(b.getSlotId());
    });
    bookingRepo.saveAll(expired);
}
```
Or use Redis TTL key as a trigger — when key expires, a listener releases the slot.

---

### 4. Idempotency — Duplicate Submissions

**Problem:** User clicks "Book" twice due to slow network. Two identical booking requests arrive. Should create one booking, not two.

**Solution:** Idempotency key — client sends a unique request ID. Server stores it and returns the same result for duplicates:
```java
@PostMapping("/bookings")
public ResponseEntity<BookingResponse> create(
    @RequestHeader("Idempotency-Key") String idempotencyKey,
    @RequestBody BookingRequest request
) {
    Optional<Booking> existing = bookingRepo.findByIdempotencyKey(idempotencyKey);
    if (existing.isPresent()) {
        return ResponseEntity.ok(toResponse(existing.get())); // return cached result
    }
    Booking booking = bookingService.create(request, idempotencyKey);
    return ResponseEntity.status(201).body(toResponse(booking));
}
```

---

### 5. Booking Status Consistency (Distributed Services)

**Problem:** Booking spans multiple services — inventory, payment, notification. If payment fails after inventory is reserved, inventory is stuck.

**Solution: Saga pattern** — each step has a compensating action:
```
Reserve capacity → Confirm payment → Send notification
       ↓ (fail)
Release capacity  ←  Compensate
```

Simple version with Spring `@Transactional` for same-DB operations:
```java
@Transactional
public Booking create(BookingRequest req) {
    capacityService.reserve(req.getVoyageId(), req.getQuantity()); // throws if no space
    Booking booking = bookingRepo.save(new Booking(req));
    notificationService.sendConfirmation(booking); // if this fails, whole tx rolls back
    return booking;
}
```
For cross-service: use an outbox pattern + message queue (Pub/Sub, Kafka) to decouple and guarantee delivery.

---

### 6. Cancellation & Amendment

**Problem:** Cancellation must release capacity, trigger refund/fee logic, update downstream systems, and notify parties — atomically.

**Key design:** State machine for booking status. Only valid transitions allowed:
```
PENDING → CONFIRMED → AMENDED
                    → CANCELLED
        → EXPIRED
```
```java
public void cancel(Long bookingId, String reason) {
    Booking booking = bookingRepo.findById(bookingId).orElseThrow();
    if (!booking.getStatus().canTransitionTo(BookingStatus.CANCELLED)) {
        throw new InvalidStatusTransitionException();
    }
    booking.setStatus(BookingStatus.CANCELLED);
    booking.setCancelReason(reason);
    capacityService.release(booking.getSlotId());
    feeService.applyNoShowFeeIfApplicable(booking);
    eventPublisher.publish(new BookingCancelledEvent(booking));
}
```

---

### 7. Notification & Cut-off Alerts

**Problem:** Operations team must be reminded of upcoming deadlines (SI cut-off, VGM cut-off, CY cut-off). Missing them = cargo rolled.

**Solution:** Scheduled jobs checking upcoming cut-offs:
```java
@Scheduled(cron = "0 0 8 * * *") // every morning at 8am
public void sendCutoffReminders() {
    LocalDate tomorrow = LocalDate.now().plusDays(1);
    List<Booking> upcoming = bookingRepo.findByCyCutoffDate(tomorrow);
    upcoming.forEach(b -> notificationService.alertCyCutoff(b));
}
```

---

### 8. High Load / Peak Season

**Problem:** Booking volume spikes during peak season — system slows or crashes.

**Solutions:**
- **Read replica** for availability checks (read-heavy)
- **Redis cache** for capacity counters instead of DB queries
- **Queue bookings** — accept request immediately, process async, notify when confirmed
- **Rate limiting** per client to prevent abuse
- **Database connection pooling** (HikariCP in Spring Boot) — tune pool size

---

### Summary: Problems & Tools

| Problem | Solution |
|---------|----------|
| Double booking | Pessimistic lock / optimistic lock / atomic DB update |
| Capacity tracking | Redis counter / atomic SQL decrement |
| Abandoned holds | Scheduled expiry job / Redis TTL |
| Duplicate requests | Idempotency key stored in DB |
| Cross-service consistency | Saga pattern / outbox + event queue |
| Invalid state changes | Booking state machine |
| Missed cut-off deadlines | Scheduled reminder jobs |
| High load | Read replica, Redis cache, async queue |
