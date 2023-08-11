# Java Concurrency In Practice - Introduction
> Created: 2023-05-26 13:50
> #book 

## History Of Concurrency

*Processes:* Isolated, independently executing programs to which the OS allocates resources.

Processes can communicate with each other through a variety of coarse-grained mechanisms: sockets, signal handlers, shared memory, semaphores, files.

Why?
+ **Resource utilization.** Programs have to wait for I/O etc. and while waiting do no useful work.
+ **Fairness.** Fine-grained time slicing for fair claims on different programs on CPU time.
+ **Convenience.** Easier or more desirable to write several programs that each perform a single task and have them coordinate.

**Threads.** Threads allow multiple streams of program control flow to coexist within a process. They share process-wide resources -- memory, file handles -- but each thread has its own program counter, stack and local variables. Multiple threads in the same program can be scheduled simultaneously on multiple CPUs.

> Threads are sometimes called *lightweight processes.* Most OSes treat threads as the basic units of scheduling.

## Benefits of Threads

+ Reduces development and maintenance costs when used properly
+ Improves the performance of complex applications.
+ Threads make it easier to model asynchronous interactions, turning them into mostly sequential ones.
+ Declutters code
+ Improves GUI responsiveness
	+ **Event dispatch thread.** When a UI event occurs, application-defined event handlers are called in the event thread.
+ Improves resource utilization and throughput
+ GC performance

> 📝 **NPTL Threads Package.** Designed to support hundreds of thousands of threads.

## Risks of Threads

### Safety Hazards

```java
@NotThreadSafe
public class UnsafeSequence {
	private int value;
	/** Returns a unique value. */
	public int getNext() {
		return value++;
	}
}
```

Two threads can call `getNext` and receive _the same value_. `value++` is not an atomic operation -- it (1) reads `value` (2) adds 1 to it in a memory register and then (3) writes out the new value. If this gets interleaved in the runtime, it is possible for both threads to see the same `value`, both add 1 to it, and both write out the same result.

> 💡 Annotate code that can be safely used in a multithreaded environment with `@ThreadSafe`.

This is a common concurrency hazard called a **race condition**.

Fix:
```java
@ThreadSafe
public class Sequence {
	@GuardedBy("this") private int value;

	public synchronized int getNext() {
		return value++;
	}
}
```

In the absence of synchronization, the compiler, hardware, and runtime are allowed to take substantial liberties with the timing and ordering of actions, such as **caching variables in registers** or processor-local caches where they are temporarily (or even permanently) **invisible to other threads**. These are in aid of better performance, but when threads are involved we need to know where data is shared so that these optimizations don't undermine safety.

### Liveness Hazards

Liveness: "something good eventually happens". A liveness failure occurs when an activity gets into a state such that it is permanently unable to make forward progress.

E.g. Thread A is waiting for a resource that Thread B holds exclusively, and B never releases it -- then A will wait forever.

Bugs that cause liveness failures can be elusive because they depend on the relative timing of events in different threads, and therefore do not always manifest themselves in development or testing. 

### Performance Hazards

While liveness means that something good eventually happens, eventually may not be good enough —we often want good things to happen quickly.

E.g. *Context switches* -- when the scheduler suspends the active thread temporarily so another thread can run. There are significant costs with saving and restoring execution context, loss of locality, CPU spent time scheduling threads instead of running. 

The synchronization mechanisms can inhibit compiler optimizations, flush memory caches, create synchronization traffic on the shared memory bus.

## Threads are everywhere

The need for thread safety is contagious: Frameworks introduce concurrency into applications by calling application components from framework threads. Components invariably access application state, thus requiring that *all* code paths accessing that state be thread-safe.

**Timer.** `Timer` is a convenience mechanism for scheduling tasks to run at a later time. If a `TimerTask` accesses data that is also accessed by other app threads, then not only must the `TimerTask` do so in a threadsafe manner, but *so must any other classes that access that data*.

**Servlets and JSPs.** The servlets specification requires that a servlet be prepared to be called simultaneously from multiple threads -- they need to be thread-safe. They also access state information shared with other servlets, such as app-scoped objects or session-scoped objects.

**Remote Method Invocation (RMI).** To invoke methods on objects running in another JVM. The method arguments are packaged (marshaled) into a byte stream and shipped over the network to a remote JVM. Here they are unpacked and passed to the remote method. The thread for this call is managed by RMI, so the remote object must guard against thread safety hazards.

**Swing and AWT.** Swing components, such as `JTable`, are not thread-safe. Swing programs achieve thread safety by confining all access to GUI components to the event thread.

----

## References
1. Book: Java Concurrency In Practice Chapter 1