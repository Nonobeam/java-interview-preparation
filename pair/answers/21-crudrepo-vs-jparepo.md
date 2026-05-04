## A25: CrudRepository vs JpaRepository Hierarchy

### Inheritance Chain

```
Repository<T, ID>            (marker interface, no methods)
   ↑
CrudRepository<T, ID>        (basic CRUD)
   ↑
PagingAndSortingRepository<T, ID>   (adds pagination/sorting)
   ↑
JpaRepository<T, ID>         (adds JPA-specific: flush, batch, examples)
```

Spring Data 3 also added:
- `ListCrudRepository<T, ID>` — same as CrudRepository but methods return `List` instead of `Iterable`
- `ListPagingAndSortingRepository<T, ID>` — same idea

### What Each Adds

**`Repository<T, ID>`** — marker interface, no methods. Rarely used directly.

**`CrudRepository<T, ID>`**
```java
<S extends T> S save(S entity);
<S extends T> Iterable<S> saveAll(Iterable<S> entities);
Optional<T> findById(ID id);
boolean existsById(ID id);
Iterable<T> findAll();
Iterable<T> findAllById(Iterable<ID> ids);
long count();
void deleteById(ID id);
void delete(T entity);
void deleteAll();
void deleteAllById(Iterable<? extends ID> ids);
```

**`ListCrudRepository<T, ID>`** (Spring Data 3+)
Same methods as CrudRepository, but return `List<T>` instead of `Iterable<T>`. Cleaner for most code since you almost always want a List.

**`PagingAndSortingRepository<T, ID>`** — adds:
```java
Iterable<T> findAll(Sort sort);
Page<T> findAll(Pageable pageable);
```
`Pageable` = page number + page size + sort. `Page<T>` contains content + total count + metadata.

**`JpaRepository<T, ID>`** — JPA-specific additions:
```java
void flush();                            // force immediate DB write
<S extends T> S saveAndFlush(S entity);
<S extends T> List<S> saveAllAndFlush(Iterable<S> entities);
void deleteAllInBatch();                 // single DELETE, faster
void deleteAllByIdInBatch(Iterable<ID> ids);
void deleteAllInBatch(Iterable<T> entities);
T getReferenceById(ID id);               // lazy proxy, no SELECT
List<T> findAll();                       // returns List, not Iterable
List<T> findAll(Sort sort);
List<T> findAllById(Iterable<ID> ids);
List<S> saveAll(Iterable<S> entities);
<S extends T> List<S> findAll(Example<S> example);  // Query-by-Example
```

### Key JpaRepository-Only Features

**1. `flush()` and `saveAndFlush()`**
Spring/JPA normally delays writes until transaction commit. `flush()` forces an immediate write to the DB (still inside the transaction). Useful when you need generated IDs or constraints checked immediately.

**2. `deleteAllInBatch()`**
Regular `deleteAll()` fetches each entity, triggers lifecycle callbacks, and deletes one at a time. `deleteAllInBatch()` runs a single `DELETE FROM table` — much faster but **bypasses JPA lifecycle callbacks and cascades**.

**3. `getReferenceById()`**
Returns a **proxy** without actually querying. No SELECT is issued. Useful when you need the entity just to set as a foreign key reference:
```java
// Save a Booking with shipperId = 42, without loading the Shipper
Shipper ref = shipperRepo.getReferenceById(42L);
booking.setShipper(ref);
bookingRepo.save(booking);  // INSERT with shipper_id = 42, no SELECT on shipper
```
Previously called `getOne()` (deprecated).

**4. Query-by-Example (`findAll(Example<T>)`)**
Pass a probe entity, get back matching entities. Less common than derived queries or `@Query`.

### When to Use Which

| Need | Use |
|------|-----|
| Simple CRUD, no pagination | `CrudRepository` (or `ListCrudRepository`) |
| Pagination + sorting | `PagingAndSortingRepository` |
| Batch operations, flush control, `getReferenceById` | `JpaRepository` |
| Maximum flexibility in modern codebase | `JpaRepository` (most common choice) |

In practice, **most teams just use `JpaRepository` everywhere** — it's a superset of the others. The more minimal interfaces exist for:
- Data store portability (using Spring Data without JPA — e.g., Mongo, Cassandra, Redis)
- Signaling intent ("this repo only needs CRUD, no pagination")

### Iterable vs List — Why It Matters

```java
Iterable<Booking> bookings = repo.findAll();  // CrudRepository
List<Booking> bookings = repo.findAll();      // JpaRepository or ListCrudRepository
```

`Iterable` is more restrictive — no `.size()`, no indexing, can't use `.stream()` directly (need `StreamSupport`). For JPA we almost always want `List`, which is why `JpaRepository` overrides these methods.

### Performance Tip — saveAll vs saveAllAndFlush

```java
repo.saveAll(entities);           // queued, written on TX commit
repo.saveAllAndFlush(entities);   // immediate DB write
```

For bulk inserts, also enable JDBC batching:
```properties
spring.jpa.properties.hibernate.jdbc.batch_size=50
spring.jpa.properties.hibernate.order_inserts=true
spring.jpa.properties.hibernate.order_updates=true
```
Without these, `saveAll()` of 1000 entities = 1000 individual INSERTs. With batching = 20 batched inserts of 50.

### Code Example

```java
// Typical usage
public interface BookingRepository extends JpaRepository<Booking, Long> {

    // Derived query
    List<Booking> findByStatusAndVesselName(BookingStatus status, String vesselName);

    // Pagination
    Page<Booking> findByShipperIdOrderByCreatedAtDesc(Long shipperId, Pageable pageable);

    // Custom query
    @Query("SELECT b FROM Booking b WHERE b.voyage.etd > :date")
    List<Booking> findFutureBookings(@Param("date") LocalDate date);

    // Modifying query
    @Modifying
    @Query("UPDATE Booking b SET b.status = :status WHERE b.id = :id")
    int updateStatus(@Param("id") Long id, @Param("status") BookingStatus status);
}
```

### Interview-Ready Answer

> "They form an inheritance chain: CrudRepository → PagingAndSortingRepository → JpaRepository. Each adds more features. CrudRepository is the basic CRUD. PagingAndSortingRepository adds `Page<T>` and `Sort`. JpaRepository adds JPA-specific features like `flush()`, `saveAndFlush()`, `deleteAllInBatch()`, and `getReferenceById()`.
>
> In practice I use `JpaRepository` for most things — it's a superset. The lighter interfaces are useful if you want to signal intent, or if you're using Spring Data with a non-JPA store like Mongo.
>
> Spring Data 3 added `ListCrudRepository` which returns `List` instead of `Iterable` — cleaner API since you almost always want a List anyway."
