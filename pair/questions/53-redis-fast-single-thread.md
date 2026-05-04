# 57. Why is Redis so fast, and how does it handle concurrency with a single thread?

Explain why Redis can handle hundreds of thousands of operations per second despite being single-threaded. Cover: in-memory storage, I/O multiplexing, the event loop, and why single-thread avoids lock contention. Also clarify what changed in Redis 6+ with threaded I/O.
