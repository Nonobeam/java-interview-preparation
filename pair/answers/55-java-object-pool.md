# 58. Java Object Pool

An object pool pre-creates expensive objects and reuses them instead of creating/destroying on every request.

---

## Why use it

Object creation can be expensive when it involves:
- Network connections (database, HTTP, JDBC)
- Thread creation
- Large buffer allocation
- SSL handshake setup

Instead of `new Connection()` on every request, borrow from pool → use → return.

---

## Simple pool with `BlockingQueue`

```java
public class ObjectPool<T> {
    private final BlockingQueue<T> pool;
    private final Supplier<T> factory;

    public ObjectPool(int size, Supplier<T> factory) {
        this.factory = factory;
        this.pool = new ArrayBlockingQueue<>(size);
        for (int i = 0; i < size; i++) {
            pool.offer(factory.get());
        }
    }

    public T borrow() throws InterruptedException {
        return pool.take();  // blocks if pool is empty
    }

    public T borrow(long timeout, TimeUnit unit)
            throws InterruptedException {
        T obj = pool.poll(timeout, unit);
        if (obj == null) throw new RuntimeException("Pool exhausted");
        return obj;
    }

    public void returnObject(T obj) {
        pool.offer(obj);  // non-blocking, always succeeds (pool sized correctly)
    }
}

// Usage
ObjectPool<ExpensiveResource> pool =
    new ObjectPool<>(10, ExpensiveResource::new);

ExpensiveResource res = pool.borrow();
try {
    res.doWork();
} finally {
    pool.returnObject(res);  // ALWAYS return in finally
}
```

`BlockingQueue` handles thread-safety — no explicit synchronization needed.

---

## Apache Commons Pool2

For production use, Commons Pool2 provides lifecycle management (validation, eviction, max-idle):

```java
public class MyObjectFactory extends BasePooledObjectFactory<MyObject> {

    @Override
    public MyObject create() {
        return new MyObject();  // expensive creation
    }

    @Override
    public PooledObject<MyObject> wrap(MyObject obj) {
        return new DefaultPooledObject<>(obj);
    }

    @Override
    public boolean validateObject(PooledObject<MyObject> p) {
        return p.getObject().isValid();  // health check before borrow
    }

    @Override
    public void destroyObject(PooledObject<MyObject> p) {
        p.getObject().close();  // cleanup on eviction
    }
}

GenericObjectPoolConfig<MyObject> config = new GenericObjectPoolConfig<>();
config.setMaxTotal(20);        // max objects
config.setMinIdle(5);          // always keep 5 ready
config.setMaxWaitMillis(3000); // throw after 3s if none available
config.setTestOnBorrow(true);  // call validateObject() before each borrow

GenericObjectPool<MyObject> pool =
    new GenericObjectPool<>(new MyObjectFactory(), config);

MyObject obj = pool.borrowObject();
try {
    obj.process();
} finally {
    pool.returnObject(obj);
}
```

---

## Real-world example: HikariCP (database connection pool)

HikariCP is a production-grade connection pool built on the same concept:

```yaml
# application.yml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20      # max connections
      minimum-idle: 5            # connections kept ready
      connection-timeout: 30000  # wait up to 30s for a connection
      idle-timeout: 600000       # close idle connections after 10m
      max-lifetime: 1800000      # recycle connections after 30m
```

JPA/JDBC borrows a connection from HikariCP per transaction, returns it on commit/rollback. You never call `new Connection()`.

---

## Thread pool = same pattern

`ExecutorService` thread pools (`Executors.newFixedThreadPool(n)`) apply the exact same pattern to threads:
- Pool of pre-created threads
- Tasks borrow a thread, run to completion, return to pool
- Avoids `new Thread()` creation cost per task

---

## Key rules

1. **Always return in `finally`** — leaked objects starve the pool
2. **Validate before use** — stale connections, expired objects
3. **Size appropriately** — too small → contention; too large → wasted resources
4. **Don't pool cheap objects** — `String`, `int`, simple POJOs don't need pooling

---

## Interview one-liner
> "An object pool pre-creates expensive objects (connections, threads, buffers) and reuses them instead of creating/destroying per request. In Java, you can build one with a `BlockingQueue` (thread-safe by design) or use Apache Commons Pool2 for lifecycle features. HikariCP is the most common real-world pool — it manages JDBC connections exactly this way."
