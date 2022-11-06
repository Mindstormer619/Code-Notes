# CoroutineScope & Job
> Created: 2022-09-01 18:03

## CoroutineScope

`interface` [CoroutineScope](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html)

Defines a scope for new coroutines. Every **coroutine builder** (like [launch](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html), [async](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html), etc.) is an extension on [CoroutineScope](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html) and inherits its [coroutineContext](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/coroutine-context.html) to automatically propagate all its elements and cancellation.

The best ways to obtain a standalone instance of the scope are CoroutineScope() and MainScope() factory functions, taking care to cancel these coroutine scopes when they are no longer needed (see section on custom usage below for explanation and example).

Additional context elements can be appended to the scope using the [plus](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/plus.html) operator.

### Properties

`abstract val` [coroutineContext](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/coroutine-context.html): [CoroutineContext](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/-coroutine-context/index.html)

The context of this scope. Context is encapsulated by the scope and used for implementation of coroutine builders that are extensions on the scope. Accessing this property in general code is not recommended for any purposes except accessing the [Job](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html) instance for advanced usages.

### Convention for structured concurrency

Manual implementation of this interface is not recommended, implementation by delegation should be preferred instead. By convention, the [context of a scope](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/coroutine-context.html) should contain an instance of a [job](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html) to enforce the discipline of **structured concurrency** with propagation of cancellation.

Every coroutine builder (like [launch](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html), [async](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html), and others) and every scoping function (like [coroutineScope](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html) and [withContext](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-context.html)) provides _its own_ scope with its own [Job](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html) instance into the inner block of code it runs. By convention, they all wait for all the coroutines inside their block to complete before completing themselves, thus enforcing the structured concurrency. See [Job](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html) documentation for more details.

## Job

`interface` [Job](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html) : [CoroutineContext.Element](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/-coroutine-context/-element/index.html)

A background job. Conceptually, a job is a cancellable thing with a life-cycle that culminates in its completion.

Jobs can be arranged into parent-child hierarchies where cancellation of a parent leads to immediate cancellation of all its [children](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/children.html) recursively. Failure of a child with an exception other than [CancellationException](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-cancellation-exception/index.html) immediately cancels its parent and, consequently, all its other children. This behavior can be customized using [SupervisorJob](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-supervisor-job.html).

The most basic instances of `Job` interface are created like this:

- **Coroutine job** is created with [launch](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html) coroutine builder. It runs a specified block of code and completes on completion of this block.
- [**CompletableJob**](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-completable-job/index.html) is created with a `Job()` factory function. It is completed by calling [CompletableJob.complete](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-completable-job/complete.html).  

Conceptually, an execution of a job does not produce a result value. Jobs are launched solely for their side-effects. See [Deferred](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/index.html) interface for a job that produces a result.

### Cancellation cause

A coroutine job is said to _complete exceptionally_ when its body throws an exception; a [CompletableJob](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-completable-job/index.html) is completed exceptionally by calling [CompletableJob.completeExceptionally](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-completable-job/complete-exceptionally.html). An exceptionally completed job is cancelled and the corresponding exception becomes the _cancellation cause_ of the job.

Normal cancellation of a job is distinguished from its failure by the type of this exception that caused its cancellation. A coroutine that threw [CancellationException](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-cancellation-exception/index.html) is considered to be _cancelled normally_. If a cancellation cause is a different exception type, then the job is considered to have _failed_. When a job has _failed_, then its parent gets cancelled with the exception of the same type, thus ensuring transparency in delegating parts of the job to its children.

Note, that [cancel](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/cancel.html) function on a job only accepts [CancellationException](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-cancellation-exception/index.html) as a cancellation cause, thus calling [cancel](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/cancel.html) always results in a normal cancellation of a job, which does not lead to cancellation of its parent. This way, a parent can [cancel](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/cancel.html) its own children (cancelling all their children recursively, too) without cancelling itself.

----

## References
1. https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/