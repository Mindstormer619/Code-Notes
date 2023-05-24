# Concurrency vs. Parallelism Using Kotlin
> Created: 2023-05-23 14:33

## Asynchrony: A programming model

[Wikipedia](https://en.wikipedia.org/wiki/Asynchrony_(computer_programming)): the occurrence of events independent of the main program flow.

Actions started asynchronously do not immediately block the program and take place _concurrently_. When programming asynchronously, we often find ourselves triggering some subroutine that immediately returns to the caller to let the main program flow continue without waiting for the subroutine’s result. Once the result is needed, you may run into two scenarios: 1) the result has been fully processed and can just be requested, or 2) you need to block your program until it is available. That is how futures or promises, known in many languages, work.

Altogether, we can describe asynchrony, defined in the domain of software engineering, as a programming model that **enables** non-blocking and concurrent programming. 

## Concurrency: It's about structure

Concurrency is not, as many people mistakenly believe, about running things “in parallel” or “at the same time”. Rob Pike describes concurrency as a “composition of independently executing tasks” and he emphasizes that concurrency really is about _structuring a program_. 

A concurrent program handles multiple tasks being in progress at the same time but not necessarily being executed simultaneously. The work on all tasks may be interleaved in some arbitrary order, similarly to the objects used by a juggler.

Concurrency is not parallelism. It tries to break tasks down. Those tasks don’t necessarily have to execute at the same time, though. Its primary goal is _structure_, not _parallelism_.

## Parallelism: It's about execution

_Parallelism_, often mistakenly used synonymously for _concurrency_, is about the simultaneous execution of multiple things. If concurrency is about structure, then parallelism is about the execution of multiple tasks. We can say that concurrency makes the use of parallelism easier, but it is not even a prerequisite since **we can have [parallelism](https://en.wikipedia.org/wiki/Parallel_computing) without concurrency**.

Conclusively, as Rob Pike describes it: “Concurrency is about dealing with lots of things at once. Parallelism is about doing lots of things at once”. You can watch his talk _Concurrency is not Parallelism_ on [YouTube](https://youtu.be/oV9rvDllKEg).

## Coroutines

Coroutines are about _concurrency_ first of all.

```kotlin
fun main() = runBlocking {
    val time = measureTimeMillis {
        val one = async { doSomethingUsefulOne() }
        val two = async { doSomethingUsefulTwo() }
        println("The answer is ${one.await() + two.await()}")
    }
    println("Completed in $time ms")
}

suspend fun doSomethingUsefulOne(): Int {
    delay(1000L)
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L)
    return 29
}
```

The example terminates in roughly 1000 milliseconds since both “somethingUseful” tasks take about 1 second each and we execute them asynchronously with the help of the `async` coroutine builder. Both tasks just use a simple non-blocking `delay` to simulate some reasonably long-running action. Let's see if the framework executes these tasks truly simultaneously. Therefore we add some log statements that tell us the threads the actions run on:

```log
[main] DEBUG logger - in runBlocking  
[main] DEBUG logger - in doSomethingUsefulOne  
[main] DEBUG logger - in doSomethingUsefulTwo
```

Since we use `runBlocking` from the main thread, it runs on the very same one. The `async` builders do not specify a separate [CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/) or [CoroutineContext](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines.experimental/-coroutine-context/index.html) and therefore also inherently run on `main`.

The example is, as correctly described, an example of concurrency, not utilizing parallelism.

## What if the coroutines are CPU-expensive?

Instead of just delaying the coroutines, we let the functions `doSomethingUseful` calculate the [next probable prime](https://docs.oracle.com/javase/7/docs/api/java/math/BigInteger.html#nextProbablePrime()) based on a randomly generated `BigInteger` which happens to be a fairly expensive task (since this calculation is based on a random it will not run in deterministic time):

```kotlin
fun doSomethingUsefulOne(): BigInteger {
    log.debug("in doSomethingUsefulOne")
    return BigInteger(1500, Random()).nextProbablePrime()
}
```

Note that the `suspend` keyword is not necessary anymore and would actually be misleading. The function does not make use of other suspending functions and blocks the calling thread for the needed time. Running the code results in the following logs:

```log
22:22:04.716 [main] DEBUG logger - in runBlocking  
22:22:04.749 [main] DEBUG logger - in doSomethingUsefulOne  
22:22:05.595 [main] DEBUG logger - Prime calculation took 844 ms  
22:22:05.602 [main] DEBUG logger - in doSomethingUsefulOne  
22:22:08.241 [main] DEBUG logger - Prime calculation took 2638 ms  
Completed in 3520 ms
```

As we can easily see, the tasks still run concurrently as in with `async` coroutines but don't execute at the same time anymore. The overall runtime is the sum of both sub-calculations (roughly). After changing the suspending code to blocking code, the result changes and we don't win any time while executing anymore.

## Making coroutines run in parallel

### `GlobalScope`

Spawning a coroutine in the `GlobalScope` means that the coroutine is not bound to any `Job` and is only limited by the lifetime of the whole application. It dispatches a coroutine on `Dispatchers.Default`, a shared thread pool. By default, the max number of threads used by this dispatcher is equal to the number of available CPU cores (but is at least 2).

```kotlin
val time = measureTimeMillis {  
    val one = GlobalScope.async { doSomethingUsefulOne() }  
    val two = GlobalScope.async { doSomethingUsefulTwo() }  
}
```


The output verifies that we now run in roughly `max(time(calc1), time(calc2))`:

```log
22:42:19.375 [main] DEBUG logger - in runBlocking  
22:42:19.393 [DefaultDispatcher-worker-1] DEBUG logger - in doSomethingUsefulOne  
22:42:19.408 [DefaultDispatcher-worker-4] DEBUG logger - in doSomethingUsefulOne  
22:42:22.640 [DefaultDispatcher-worker-1] DEBUG logger - Prime calculation took 3245 ms  
22:42:23.330 [DefaultDispatcher-worker-4] DEBUG logger - Prime calculation took 3922 ms  
Completed in 3950 ms
```

### Specify a coroutine dispatcher

```kotlin
val time = measureTimeMillis {  
    val one = async(Dispatchers.Default) { doSomethingUsefulOne() }  
    val two = async(Dispatchers.Default) { doSomethingUsefulTwo() }  
    println("The answer is ${one.await() + two.await()}")  
}
```

We get the same result as before, while not losing the parent-child structure that we want (which allows structured concurrency). However, we can do even better and make the blocking function suspending instead.

### Make blocking function suspending

Using `withContext`: "immediately applies dispatcher from the new context, shifting execution of the block into the different thread inside the block, and back when it completes"

```kotlin
suspend fun doSomethingUsefulOne(): BigInteger = 
	withContext(Dispatchers.Default) {
	    measureTimedValue {
	        println("in doSomethingUsefulOne")
	        BigInteger(1500, Random()).nextProbablePrime()
	    }
	}.also {
	    println("Prime calculation took ${it.duration} ms")
	}.value
```

With this approach, we confine the execution of dispatched tasks to the prime calculation inside the suspending function. The output nicely demonstrates that only the actual prime calculation happens on a different thread while everything else stays on main.

```log
23:00:20.591 [main] DEBUG logger - in runBlocking  
23:00:20.648 [DefaultDispatcher-worker-1] DEBUG logger - in doSomethingUsefulOne  
23:00:20.714 [DefaultDispatcher-worker-2] DEBUG logger - in doSomethingUsefulOne  
23:00:21.132 [main] DEBUG logger - Prime calculation took 413 ms  
23:00:23.971 [main] DEBUG logger - Prime calculation took 3322 ms  
Completed in 3371 ms
```



----

## References
1. [[Fleeting/Concurrency vs. Parallelism Using Kotlin|Concurrency vs. Parallelism Using Kotlin]]