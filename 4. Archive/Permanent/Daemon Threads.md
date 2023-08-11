# Daemon Threads
> Created: 2022-09-01 17:52

Java has two types of threads -- User Threads and Daemon Threads. The JVM waits for any user thread to complete its task before terminating it.

Daemon threads, on the other hand, are _low priority threads_ that provide services to user threads. The JVM might exit once all user threads have finished their execution, while daemon threads might still be running.

**Do not use Daemon Threads for I/O tasks!** The JVM might exit before execution is finished, including `finally` blocks.

> Daemon threads are useful for background supporting tasks such as garbage collection, releasing memory of unused objects and removing unwanted entries from the cache. Most of the JVM threads are daemon threads.

## Creating a Daemon Thread

```java
NewThread daemonThread = new NewThread();
daemonThread.setDaemon(true);
daemonThread.start();
```


----

## References
1. [[2. Areas/Java/Daemon Threads]]