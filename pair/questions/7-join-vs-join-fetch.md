## Q7: What is the difference between JOIN and JOIN FETCH in JPQL?

JOIN is only for filtering — the related entity is not loaded. JOIN FETCH joins AND loads the entity into memory. Also cover: the pagination pitfall with JOIN FETCH (in-memory pagination warning).
