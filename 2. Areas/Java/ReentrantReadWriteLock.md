---
created: 2024-09-04 19:53
alias: 
---

> [!summary]-
> + 

# ReentrantReadWriteLock

## Reentrancy

Threads can re-enter into a lock on a resource multiple times without a deadlock situation. When `lock()` is requested, the hold count of the lock increases by 1. `unlock()` decreases the hold count by 1. A thread needs to call `unlock()` the same number of times as it calls `lock()` in order to release the resource to other threads.

If the owner thread of a reentrant lock goes into sleep or infinite wait, it won’t be possible to release the resource, and a deadlock situation will result.

## Code Example

```java
class CachedData {
   Object data;
   volatile boolean cacheValid;
   final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();

   void processCachedData() {
     rwl.readLock().lock();
     if (!cacheValid) {
       // Must release read lock before acquiring write lock
       rwl.readLock().unlock();
       rwl.writeLock().lock();
       try {
         // Recheck state because another thread might have
         // acquired write lock and changed state before we did.
         if (!cacheValid) {
           data = ...
           cacheValid = true;
         }
         // Downgrade by acquiring read lock before releasing write lock
         rwl.readLock().lock();
       } finally {
         rwl.writeLock().unlock(); // Unlock write, still hold read
       }
     }

     try {
       use(data);
     } finally {
       rwl.readLock().unlock();
     }
   }
 }
```

----

## References
+ https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReentrantReadWriteLock.html
+ https://www.baeldung.com/java-binary-semaphore-vs-reentrant-lock