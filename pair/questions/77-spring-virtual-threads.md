# Q77 — Spring Virtual Threads (Project Loom)

## Questions

1. What is a platform (OS) thread vs a virtual thread? What problem do virtual threads solve?
2. In traditional Spring MVC, one HTTP request = one platform thread. What happens to that thread while waiting on a DB call or HTTP call? Why is this wasteful?
3. Virtual threads are "cheap" — how cheap? How many can the JVM run simultaneously compared to platform threads?
4. How do you enable virtual threads in Spring Boot 3.2+? (one property or one bean)
5. What is "pinning" in virtual threads? When does it happen, and why is it a problem? Name two common causes (synchronized blocks, native calls).
6. Virtual threads vs reactive (WebFlux): when would you choose one over the other in a new project?
7. Does using virtual threads change your code? Do you still use `@Async`, `ExecutorService`, `CompletableFuture`?
