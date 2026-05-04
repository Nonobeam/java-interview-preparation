# 66. Spring Data JPA: derived queries, `@Query`, pagination

## 1. Derived (method-name) queries

Spring Data parses the method name and **generates JPQL automatically** at startup.

```java
public interface UserRepository extends JpaRepository<User, Long> {

    Optional<User> findByEmail(String email);

    List<User> findByStatusAndCreatedAtAfter(Status status, Instant after);

    long countByStatus(Status status);

    boolean existsByEmail(String email);

    List<User> findByEmailContainingIgnoreCaseOrderByCreatedAtDesc(String fragment);

    void deleteByStatus(Status status);  // needs @Transactional in caller
}
```

Naming grammar (the parts you'll actually use):
- Verbs: `findBy`, `getBy`, `readBy`, `countBy`, `existsBy`, `deleteBy`.
- Conditions: `And`, `Or`, `Between`, `LessThan`, `GreaterThan`, `Like`, `Containing`, `StartingWith`, `In`, `IsNull`, `IsNotNull`.
- Modifiers: `IgnoreCase`, `OrderBy<Field>Asc/Desc`, `Distinct`, `First<N>`, `Top<N>`.

**Pros:** Zero SQL/JPQL to write; refactor-safe for entity field renames (caught at startup).
**Cons:** Method names get unreadable past 2-3 conditions. Switch to `@Query` then.

## 2. `@Query` — explicit JPQL or native SQL

When the derived name would be too long, or you need a join that's awkward to express by name.

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    // JPQL — references entity names and fields
    @Query("""
        SELECT o FROM Order o
        JOIN FETCH o.items
        WHERE o.customer.id = :customerId
          AND o.createdAt >= :since
    """)
    List<Order> findRecentOrdersWithItems(@Param("customerId") Long customerId,
                                          @Param("since") Instant since);

    // Native SQL — when JPQL can't express it (window functions, vendor-specific)
    @Query(value = """
        SELECT id, customer_id, total
        FROM orders
        WHERE total > (SELECT AVG(total) FROM orders WHERE customer_id = :cid)
    """, nativeQuery = true)
    List<Order> aboveAverageForCustomer(@Param("cid") Long cid);

    // Modifying query — UPDATE / DELETE
    @Modifying
    @Query("UPDATE Order o SET o.status = :status WHERE o.id = :id")
    int updateStatus(@Param("id") Long id, @Param("status") OrderStatus status);
}
```

`@Modifying` is required for `UPDATE`/`DELETE`/`INSERT`. The caller must run inside a transaction.

`@Param` matches the named parameter; you can also use positional `?1`, `?2`.

## 3. Pagination & sorting

Spring Data's `Pageable` is the idiomatic way.

```java
public interface UserRepository extends JpaRepository<User, Long> {
    Page<User> findByStatus(Status status, Pageable pageable);
}
```

```java
// Service / controller
Page<User> page = userRepo.findByStatus(
    Status.ACTIVE,
    PageRequest.of(0, 20, Sort.by("createdAt").descending())
);

page.getContent();        // List<User>, the 20 rows on this page
page.getNumber();         // 0
page.getTotalElements();  // 1234 (runs a separate COUNT query)
page.getTotalPages();     // 62
page.hasNext();           // true
```

### `Page` vs `Slice` vs `List`

| Return type | Extra query? | Use when |
|-------------|--------------|----------|
| `Page<T>`   | Yes — COUNT for total | UI needs page numbers / total count |
| `Slice<T>`  | No — fetches `pageSize+1` to know if there's a next | Infinite scroll; only need "next exists?" |
| `List<T>`   | No                | You just want the rows; no paging metadata |

`Slice` is much cheaper than `Page` on large tables — skip the COUNT if you can.

### Pageable + `@Query`

Works the same; Spring appends the pagination clauses.

```java
@Query("SELECT o FROM Order o WHERE o.status = :status")
Page<Order> findByStatus(@Param("status") OrderStatus status, Pageable pageable);
```

For **native** queries with `Page<T>`, you must also provide a `countQuery` (Spring can't auto-generate the COUNT for native SQL):

```java
@Query(value = "SELECT * FROM orders WHERE status = :status",
       countQuery = "SELECT COUNT(*) FROM orders WHERE status = :status",
       nativeQuery = true)
Page<Order> findByStatusNative(@Param("status") String status, Pageable pageable);
```

## 4. Controller-level pagination

Spring MVC binds `Pageable` from query string out of the box (`?page=2&size=20&sort=createdAt,desc`):

```java
@GetMapping("/orders")
public Page<Order> list(@PageableDefault(size = 20) Pageable pageable) {
    return orderRepo.findAll(pageable);
}
```

## Interview one-liner

> "Derived queries let Spring generate JPQL from the method name. `@Query` is the escape hatch for complex JPQL or native SQL — use `@Modifying` for writes. For pagination, accept `Pageable` and return `Page<T>` when you need totals or `Slice<T>` for infinite-scroll UIs to avoid the COUNT cost."
