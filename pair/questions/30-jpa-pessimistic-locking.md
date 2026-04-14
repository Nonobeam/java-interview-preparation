# Q30 — JPA Pessimistic Locking

Existing Q20 covered optimistic `@Version`. This one is the pessimistic counterpart.

```java
@Entity
public class Account {
    @Id Long id;
    BigDecimal balance;
}

public interface AccountRepo extends JpaRepository<Account, Long> {

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("select a from Account a where a.id = :id")
    Account findForUpdate(@Param("id") Long id);
}

@Service
public class TransferService {
    @Transactional
    public void transfer(Long fromId, Long toId, BigDecimal amount) {
        Account from = repo.findForUpdate(fromId);
        Account to   = repo.findForUpdate(toId);
        from.setBalance(from.getBalance().subtract(amount));
        to.setBalance(to.getBalance().add(amount));
    }
}
```

## Questions

1. Contrast `LockModeType.PESSIMISTIC_READ`, `PESSIMISTIC_WRITE`, and `PESSIMISTIC_FORCE_INCREMENT`. What SQL does each generate on Oracle, and what concurrency anomalies does each prevent?
2. When would you pick pessimistic locking over `@Version`? Give two real scenarios where optimistic is the wrong choice.
3. The `transfer` method above has a deadlock risk. Describe the exact interleaving and how you'd eliminate it without dropping the locks.
4. `javax.persistence.lock.timeout` hint — what does it do on Oracle vs. PostgreSQL vs. MySQL/InnoDB? What happens when it fires?
5. If you call `repo.findForUpdate(id)` outside a transaction, what happens? What if you call it inside a `@Transactional(readOnly = true)` method?
6. Pessimistic lock + JPA second-level cache: does the cache serve stale reads around a locked row? What about the query cache?
