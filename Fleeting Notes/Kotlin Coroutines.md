# Kotlin Coroutines
> Created: 2022-05-03 23:37

## Basics

A coroutine is not bound to any particular [[kotlin.concurrent.thread|thread]]. It may suspend its execution in one thread and resume in another one.

```kotlin
fun main() = runBlocking { // this: CoroutineScope
    launch { // launch a new coroutine and continue
        delay(1000L) // non-blocking delay for 1 second (default time unit is ms)
        println("World!") // print after delay
    }
    println("Hello") // main coroutine continues while a previous one is delayed
}
```

+ `runBlocking`: Transition point between coroutine scope and regular code. Blocks the current thread
+ `launch`: _Coroutine builder_. It launches a new coroutine concurrently with the rest of the code, which continues to work independently.

### coroutineScope

[`coroutineScope`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html) builder. It creates a coroutine scope and does not complete until all launched children complete.

[`runBlocking`][rb] and `coroutineScope` builders may look similar because they both wait for their body and all its children to complete. The main difference is that the `runBlocking` method _blocks_ the current thread for waiting, while `coroutineScope` just suspends, releasing the underlying thread for other usages. Because of that difference, `runBlocking` is a regular function and `coroutineScope` is a suspending function.

### Wait for job completion

```kotlin
val job = launch { // launch a new coroutine and keep a reference to its Job
    delay(1000L)
    println("World!")
}
println("Hello")
job.join() // wait until child coroutine completes
println("Done") 
```




----

## References
1. https://kotlinlang.org/docs/coroutines-guide.html

[rb]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-blocking.html