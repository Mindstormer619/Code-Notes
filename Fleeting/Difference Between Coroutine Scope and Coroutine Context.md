# Difference Between Coroutine Scope and Coroutine Context
> Created: 2023-05-21 22:50

In brief, the coroutine context is a holder of data related to the coroutine, while the coroutine scope is a holder of the coroutine context.

## Coroutine Scope

To launch a coroutine, we need to use a coroutine builder like _launch_ or _async_. These builder functions are actually extensions of the _CoroutineScope_ interface. So, whenever we want to launch a coroutine, we need to start it in some scope.

The scope creates relationships between coroutines inside it and allows us to manage the lifecycles of these coroutines. There are several scopes provided by the _kotlinx.coroutines_ library that we can use when launching a coroutine.

### _GlobalScope_

One of the simplest ways to run a coroutine is to use _GlobalScope_:

```kotlin
GlobalScope.launch {
    delay(500L)
    println("Coroutine launched from GlobalScope")
}
```

The lifecycle of this scope is tied to the lifecycle of the whole application. This means that the scope will stop running either after all of its coroutines have been completed or when the application is stopped.

It’s worth mentioning that coroutines launched using _GlobalScope_ do not keep the process alive. They behave similarly to daemon threads. So, even when the application stops, some active coroutines will still be running. This can easily create resource or memory leaks.

### _runBlocking_

Another scope that comes right out of the box is _runBlocking_. From the name, we might guess that it creates a scope and runs a coroutine in a blocking way. This means it blocks the current thread until all childrens’ coroutines complete their executions.

**It is not recommended to use this scope because threads are expensive and will depreciate all the benefits of coroutines**.

The most suitable place for using _runBlocking_ is the very top level of the application, which is the _main_ function. Using it in _main_ will ensure that the app will wait until all child jobs inside _runBlocking_ complete.

Another place where this scope fits nicely is in tests that access suspending functions.

### _coroutineScope_

For all the cases when we don’t need thread blocking, we can use _coroutineScope_. Similarly to _runBlocking_, it will wait for its children to complete. But unlike _runBlocking_, this scope doesn’t block the current thread but only suspends it because _coroutineScope_ is a suspending function.

Consider reading our companion article to find more details about the [differences between _runBlocking_ and _coroutineScope_](https://www.baeldung.com/kotlin/coroutines-runblocking-coroutinescope).

### Custom Coroutine Scope

There might be cases when we need to have some specific behavior of the scope to get a different approach in managing the coroutines. To achieve that, we can implement the _CoroutineScope_ interface and implement our custom scope for coroutine handling.

## Coroutine Context

Now, let’s take a look at the role of _CoroutineContext_ here. The context is a holder of data that is needed for the coroutine. Basically, it’s an indexed set of elements where each element in the set has a unique key.

The important elements of the coroutine context are the _Job_ of the coroutine and the _Dispatcher_.

Kotlin provides an easy way to add these elements to the coroutine context using the “_+”_ operator:

```kotlin
launch(Dispatchers.Default + Job()) {
    println("Coroutine works in thread ${Thread.currentThread().name}")
}
```

### _Job_ in the Context[](https://www.baeldung.com/kotlin/coroutines-scope-vs-context#1-job-in-the-context)

A _Job_ of a coroutine is to handle the launched coroutine. For example, it can be used to wait for coroutine completion explicitly.

Since _Job_ is a part of the coroutine context, it can be accessed using the `coroutineContext[Job]` expression.

### 3.2. Coroutine Context and Dispatchers[](https://www.baeldung.com/kotlin/coroutines-scope-vs-context#2-coroutine-context-and-dispatchers)

Another important element of the context is _Dispatcher_. It determines what threads the coroutine will use for its execution.

Kotlin provides several implementations of _CoroutineDispatcher_ that we can pass to the _CoroutineContext_:

-   _Dispatchers.Default_ uses a shared thread pool on the JVM. By default, the number of threads is equal to the number of CPUs available on the machine.
-   _Dispatchers.IO_ is designed to offload blocking IO operations to a shared thread pool.
-   _Dispatchers.Main_ is present only on platforms that have main threads, such as Android and iOS.
-   _Dispatchers.Unconfined_ doesn’t change the thread and launches the coroutine in the caller thread. The important thing here is that after suspension, it resumes the coroutine in the thread that was determined by the suspending function.

### Switching the Context[](https://www.baeldung.com/kotlin/coroutines-scope-vs-context#3-switching-the-context)

Sometimes, we must change the context during coroutine execution while staying in the same coroutine. We can do this **using the _withContext_ function**. It will call the specified suspending block with a given coroutine context. The outer coroutine suspends until this block completes and returns the result:

```kotlin
newSingleThreadContext("Context 1").use { ctx1 ->
    newSingleThreadContext("Context 2").use { ctx2 ->
        runBlocking(ctx1) {
            println("Coroutine started in thread from ${Thread.currentThread().name}")
            withContext(ctx2) {
                println("Coroutine works in thread from ${Thread.currentThread().name}")
            }
            println("Coroutine switched back to thread from ${Thread.currentThread().name}")
        }
    }
}
```

The context of the _withContext_ block will be the merged contexts of the coroutine and the context passed to _withContext_.

### **Children of a Coroutine**[](https://www.baeldung.com/kotlin/coroutines-scope-vs-context#4-children-of-a-coroutine)

When we launch a coroutine inside another coroutine, it inherits the outer coroutine’s context, and the job of the new coroutine becomes a child job of the parent coroutine’s job. Cancellation of the parent coroutine leads to cancellation of the child coroutine as well.

We can override this parent-child relationship using one of two ways:
-   Explicitly specify a different scope when launching a new coroutine
-   Pass a different _Job_ object to the context of the new coroutine

In both cases, the new coroutine will not be bound to the scope of the parent coroutine. It will execute independently, meaning that canceling the parent coroutine won’t affect the new coroutine.

----

## References
1. https://www.baeldung.com/kotlin/coroutines-scope-vs-context#overview