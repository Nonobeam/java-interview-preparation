# 55. Java Threading

Java threading is how you run code concurrently. The core challenge is coordinating shared mutable state between threads.

---

## Creating threads

```java
// 1. Extend Thread
class MyThread extends Thread {
    public void run() { System.out.println("running"); }
}
new MyThread().start();

// 2. Implement Runnable (preferred — no return value, no checked exception)
Thread t = new Thread(() -> System.out.println("running"));
t.start();

// 3. Callable — returns a value, can throw checked exception
Callable<Integer> task = () -> {
    // do work
    return 42;
};
```

---

## ExecutorService (prefer over raw threads)

Raw `new Thread()` is wasteful — thread creation is expensive. Use a thread pool:

```java
// Fixed pool — good for known concurrency level
ExecutorService pool = Executors.newFixedThreadPool(4);

// Submit Runnable (no return)
pool.execute(() -> processOrder(order));

// Submit Callable (returns Future)
Future<Integer> future = pool.submit(() -> heavyComputation());
Integer result = future.get();           // blocks until done
Integer result = future.get(5, SECONDS); // with timeout

// Always shut down when done
pool.shutdown();                          // graceful: wait for running tasks
pool.shutdownNow();                       // interrupt running tasks
```

Common pool types:
```java
Executors.newFixedThreadPool(n)     // n threads, queue for excess tasks
Executors.newCachedThreadPool()     // grows as needed, recycles idle threads (short tasks)
Executors.newSingleThreadExecutor() // one thread, tasks in order
Executors.newScheduledThreadPool(n) // for delayed / periodic tasks
```

---

## CompletableFuture (modern async)

```java
CompletableFuture<Order> future = CompletableFuture
    .supplyAsync(() -> orderRepo.findById(id))          // runs in ForkJoinPool
    .thenApply(order -> enrichWithInventory(order))     // transforms result
    .thenApply(order -> enrichWithPricing(order))
    .exceptionally(ex -> Order.empty());                // fallback on error

Order result = future.join();  // block and get result

// Combine two futures
CompletableFuture<Inventory> inv    = fetchInventory(productId);
CompletableFuture<Pricing>   price  = fetchPricing(productId);
CompletableFuture<ProductDetail> combined = inv.thenCombine(price,
    (i, p) -> new ProductDetail(i, p));
```

---

## synchronized — mutual exclusion

Prevents two threads from executing the same block simultaneously.

```java
public class Counter {
    private int count = 0;

    // Synchronized method — locks on 'this'
    public synchronized void increment() {
        count++;
    }

    // Synchronized block — finer granularity
    public void incrementBlock() {
        synchronized (this) {
            count++;
        }
    }
}
```

`count++` is not atomic — it's read → modify → write. Without synchronization, two threads can interleave and lose an increment.

---

## volatile — visibility guarantee

Ensures a variable's value is read/written to main memory, not a thread-local CPU cache.

```java
public class StatusChecker {
    private volatile boolean running = true;

    public void stop() { running = false; }     // written by one thread

    public void run() {
        while (running) {                        // read by another thread
            doWork();
        }
    }
}
```

`volatile` guarantees **visibility** but NOT atomicity. Use it only for simple flags. For compound operations (check-then-act), use `synchronized` or `AtomicXxx`.

---

## Atomic classes

```java
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();              // atomic read-modify-write
counter.compareAndSet(expected, newVal); // CAS — no lock, spin-based
```

`AtomicInteger`, `AtomicLong`, `AtomicReference`, `AtomicBoolean` — lock-free, thread-safe for single-variable operations.

---

## Common synchronizers

```java
// CountDownLatch — wait for N events to complete
CountDownLatch latch = new CountDownLatch(3);
executor.submit(() -> { doWork(); latch.countDown(); });
executor.submit(() -> { doWork(); latch.countDown(); });
executor.submit(() -> { doWork(); latch.countDown(); });
latch.await();  // blocks until count reaches 0

// Semaphore — limit concurrent access
Semaphore sem = new Semaphore(5);   // max 5 concurrent
sem.acquire();
try { accessResource(); } finally { sem.release(); }

// CyclicBarrier — all threads wait at a point, then proceed together
CyclicBarrier barrier = new CyclicBarrier(3);
// each thread calls barrier.await() — all three resume together
```

---

## Thread-safe collections

```java
// Concurrent map — lock striping (much better than Collections.synchronizedMap)
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

// Blocking queues — for producer/consumer
BlockingQueue<Task> queue = new LinkedBlockingQueue<>(100);
queue.put(task);          // blocks if full
Task t = queue.take();    // blocks if empty

// CopyOnWriteArrayList — reads without lock, writes create new array
// Good for read-heavy, write-rare scenarios (event listener lists)
```

---

## Common pitfalls

| Problem | Symptom | Fix |
|---|---|---|
| **Race condition** | Inconsistent results under load | `synchronized` / `AtomicXxx` |
| **Deadlock** | Threads wait for each other forever | Consistent lock ordering, timeout |
| **Visibility bug** | Thread reads stale cached value | `volatile` / `synchronized` |
| **Thread starvation** | Low-priority thread never runs | Fair locks (`ReentrantLock(true)`) |

---

## Interview one-liner
> "Raw threads are expensive; prefer `ExecutorService` thread pools and `CompletableFuture` for async work. Use `synchronized` for mutual exclusion (prevents race conditions), `volatile` for visibility of simple flags (not atomicity), and `AtomicXxx` for lock-free single-variable operations. `ConcurrentHashMap` and `BlockingQueue` are thread-safe collection choices for concurrent producers/consumers."
