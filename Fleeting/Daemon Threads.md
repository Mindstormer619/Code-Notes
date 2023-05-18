# Daemon Threads
> Created: 2022-05-07 00:31

Java offers two types of threads: user threads and daemon threads.

User threads are high-priority threads. **The JVM will wait for any user thread to complete its task before terminating it.**

On the other hand, **daemon threads are low-priority threads whose only role is to provide services to user threads.**

Since daemon threads are meant to serve user threads and are only needed while user threads are running, they won't prevent the JVM from exiting once all user threads have finished their execution.

That's why infinite loops, which typically exist in daemon threads, will not cause problems, because any code, including the _finally_ blocks, won't be executed once all user threads have finished their execution. For this reason, **daemon threads are not recommended for I/O tasks.**

However, there're exceptions to this rule. Poorly designed code in daemon threads can prevent the JVM from exiting. For example, calling `Thread.join()` on a running daemon thread can block the shutdown of the application.

> Daemon threads are useful for background supporting tasks such as garbage collection, releasing memory of unused objects and removing unwanted entries from the cache. Most of the JVM threads are daemon threads.

----

## References
1. https://www.baeldung.com/java-daemon-thread