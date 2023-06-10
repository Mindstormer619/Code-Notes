# Kotlin & Java Concurrency Structures
> Created: 2023-06-10 18:20

## Lock vs Synchronized Block

**A `synchronized` block is fully contained within a method.** We can have _Lock_ APIs `lock()` and `unlock()` operation in separate methods.

Synchronized block does not support fairness -- any thread can acquire a lock, causing starvation. `Lock` APIs allow `fairness` property to make the longest waiting thread have access to the lock.

Thread gets blocked if it can't get access to a synchronized block. The `Lock` API provides `tryLock()`, where the thread acquires the lock only if it's available and not held by another thread.

A thread in *waiting* state to acquire access to the synchronized block cannot be interrupted. `Lock` API gives a method `lockInterruptibly()` that can be used to interrupt the thread when it's waiting for the lock.

#todo 

----

## References
1. https://www.baeldung.com/java-concurrent-locks
2. https://proandroiddev.com/synchronization-and-thread-safety-techniques-in-java-and-kotlin-f63506370e6d
3. https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/Lock.html