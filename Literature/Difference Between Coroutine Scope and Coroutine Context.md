# Difference Between Coroutine Scope and Coroutine Context
> Created: 2023-05-21 22:56

## 1. Overview

Context: Holder of data related to the coroutine.
Scope: Holder of the context itself.

## 2. Coroutine Scope

To launch a coroutine, we use `launch` or `async`, which are coroutine builder functions **extensions of the `CoroutineScope` interface**. Launching a coroutine **always happens in some scope**.

Scope:
+ Creates relationships between coroutines inside.
+ Allows us to manage the lifecycles of those coroutines.

### 2.1 `GlobalScope`

```kotlin
GlobalScope.launch {
    delay(500L)
    println("Coroutine launched from GlobalScope")
}
```

Lifecycle of this scope tied to lifecycle of the whole application. This means that the scope will stop running either after all of its coroutines have been completed or when the application is stopped.

They behave similarly to daemon threads. So, even when the application stops, some active coroutines will still be running. **This can easily create resource or memory leaks**.

### 2.2 `runBlocking`

Creates a scope and runs the coroutine in a *blocking way*. **Blocks the current thread until all children's coroutines are completed.**

**It is not recommended to use this scope because threads are expensive and will depreciate all the benefits of coroutines**. Generally it is  only used at the very top level of the app, which is (usually) `main`. Or use it in tests that access suspending functions.

### 2.3 `coroutineScope`

Similar to `runBlocking`, will wait for all children to complete. But _does not block_ the underlying thread -- it suspends it. `coroutineScope` is a suspending function.

### 2.4 Custom Coroutine Scope

Implement the `CoroutineScope` interface.

## 3. Coroutine Context

Context is the holder of data that is needed for the coroutine -- an indexed set of elements where each element in the set has a unique key.

Important elements of the context:
+ The `Job` of the coroutine
+ The `Dispatcher`

Easy way to add these elements to the coroutine context using the `+` operator:

```kotlin
launch(Dispatchers.Default + Job()) {
    println("Coroutine works in thread ${Thread.currentThread().name}")
}
```

### 3.1 `Job` in the Context

The `Job` is an object that can be used to handle the launched coroutine -- for example, to wait for the coroutine completion explicitly.

> ❓ Is that the `await` pattern basically?

### 3.2 Dispatchers

Determines what threads the coroutine will use for its execution.

Kotlin provides implementations:
+ `Default` uses a shared thread pool on the JVM -- no of threads equal to no of CPUs by default.
+ `IO` for offloading blocking IO ops to a shared thread pool
+ `Main` present only on platforms that have main threads (Android, iOS)
+ `Unconfined` doesn't change the thread and launches the coroutine in the caller thread. After suspension, it resumes the coroutine in the thread that was determined by the suspending function.

### 3.3 Switching the context

During coroutine execution, we can change context while staying in the same coroutine, **using the `withContext` function**. Calls the specified suspending block with a given coroutine context. The outer coroutine suspends until the block completes and returns result.

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

Context of the `withContext` block will be the **merged contents** of the coroutine and the context passed to `withContext`.

### 3.4 Children of a Coroutine

When we launch one coroutine inside another, it inherits the outer coroutine's context, and the job of the new coroutine becomes a **child job** of the parent coroutine's job. Cancellation of parent leads to cancellation of child.

To override this parent-child relationship:
+ Explicitly specify a different scope when launching a new coroutine
+ OR pass a different `Job` object to the context of the new coroutine

## 4. Example from StackOverflow (ref 2)

```kotlin
runBlocking {
    val scope0 = this
    // scope0 is the top-level coroutine scope.
    scope0.launch {
        val scope1 = this
        // scope1 inherits its context from scope0. It replaces the Job field
        // with its own job, which is a child of the job in scope0.
        // It retains the Dispatcher field so the launched coroutine uses
        // the dispatcher created by runBlocking.
        scope1.launch {
            val scope2 = this
            // scope2 inherits from scope1
        }
    }
}
```



----

## References
1. [[Fleeting/Difference Between Coroutine Scope and Coroutine Context|Difference Between Coroutine Scope and Coroutine Context]]
2. https://stackoverflow.com/a/54418265/3477606