# Kotlin & Java Concurrency Structures
> Created: 2023-06-10 18:20

## Lock vs Synchronized Block

**A `synchronized` block is fully contained within a method.** We can have _Lock_ APIs `lock()` and `unlock()` operation in separate methods.

Synchronized block does not support fairness -- any thread can acquire a lock, causing starvation. `Lock` APIs allow `fairness` property to make the longest waiting thread have access to the lock.

Thread gets blocked if it can't get access to a synchronized block. The `Lock` API provides `tryLock()`, where the thread acquires the lock only if it's available and not held by another thread.

A thread in *waiting* state to acquire access to the synchronized block cannot be interrupted. `Lock` API gives a method `lockInterruptibly()` that can be used to interrupt the thread when it's waiting for the lock.

Code block for locking:
```java
Lock lock = ...; 
]();
try {
    // access to the shared resource
} finally {
    lock.unlock();
}
```

> 💡 **Call `lock()` outside the `try` block!** The reason for this is that the `lock()` call might fail with an *unchecked* exception, in which case the lock was never acquired in the first place. Hence the call to `lock.unlock()` inside `finally` *might* release a lock that was never acquired by the current thread -- i.e. someone else's lock. Keeping the `lock()` call before the `try` block starts ensures that if the lock fails to acquire, it *does not run* the code inside `finally`. See more in [this Stackoverflow answer](https://stackoverflow.com/a/31058774/3477606).

Lock interface:

```java
package java.util.concurrent.locks;  

public interface Lock {  
  
/**  
* Acquires the lock, blocking the thread.
*/  
void lock();  
  
/**  
* Acquires the lock, but the thread acquiring can be interrupted
* through a thrown InterruptedException.
*/  
void lockInterruptibly() throws InterruptedException;  
  
/**  
* Nonblocking -- attempts to acquire the lock immediately, returns
* true if locking succeeds.
*/  
boolean tryLock();  
  
/**  
* Similar to `tryLock`, but waits for {time} units before giving up.
*/  
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;  
  
/**  
* Releases the lock.
*/  
void unlock();  

Condition newCondition();  
}
```

## Lock Implementations

### ReentrantLock

When a thread reenters the lock, it has to request for the unlock the same number of times to release the resource:
```java
reentrantLock.lock();
reentrantLock.lock();
assertEquals(2, reentrantLock.getHoldCount());
assertEquals(true, reentrantLock.isLocked());

reentrantLock.unlock();
assertEquals(1, reentrantLock.getHoldCount());
assertEquals(true, reentrantLock.isLocked());

reentrantLock.unlock();
assertEquals(0, reentrantLock.getHoldCount());
assertEquals(false, reentrantLock.isLocked());
```

### ReentrantReadWriteLock

Implements the `ReadWriteLock` interface.

+ **Read lock.** If no thread acquired/requested the write lock, multiple threads can acquire the read lock.
+ **Write lock.** If no threads are reading or writing, only one thread can acquire the write lock.




----

## References
1. https://www.baeldung.com/java-concurrent-locks
2. https://proandroiddev.com/synchronization-and-thread-safety-techniques-in-java-and-kotlin-f63506370e6d
3. https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/Lock.html
4. https://www.baeldung.com/java-binary-semaphore-vs-reentrant-lock