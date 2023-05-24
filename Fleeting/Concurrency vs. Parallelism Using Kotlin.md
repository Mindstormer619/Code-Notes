# Concurrency vs. Parallelism Using Kotlin
> Created: 2023-05-23 14:19

In this article, we will see that coroutines are mostly concerned about concurrency and not primarily about parallelism. Coroutines provide sophisticated means for helping us structure code in order to make it highly concurrently executable. In addition, they allow enabling parallelism, which isn’t the default behavior necessarily.

# Asynchrony — A programming model

Asynchronous programming is a topic we’ve been reading and hearing about a lot in the last couple of years. It mainly refers to “the occurrence of events independent of the main program flow” and also “ways to deal with these events” ([Wikipedia](https://en.wikipedia.org/wiki/Asynchrony_(computer_programming))). One crucial aspect of asynchronous programming is the fact that actions started asynchronously do not immediately block the program and take place _concurrently_. When programming asynchronously, we often find ourselves triggering some subroutine that immediately returns to the caller to let the main program flow continue without waiting for the subroutine’s result. Once the result is needed, you may run into two scenarios: 1) the result has been fully processed and can just be requested, or 2) you need to block your program until it is available. That is how futures or promises, known in many languages, work. Another popular example of asynchrony is described in this part of the [Reactive Manifesto](https://www.reactivemanifesto.org/) covering the use of reactive streams:

> _Reactive Systems rely on_ asynchronous message-passing _to establish a boundary between components that ensures loose coupling, isolation and location transparency. {...} Non-blocking communication allows recipients to only consume resources while active, leading to less system overhead._

Altogether, we can describe asynchrony, defined in the domain of software engineering, as a programming model that **enables** non-blocking and concurrent programming. We dispatch tasks to let our program continue doing something else until we receive a signal that the results are available.

# Concurrency — It’s about structure

After we learned what asynchrony refers to, let’s see what _concurrency_ is. Concurrency is not, as many people mistakenly believe, about running things “in parallel” or “at the same time”. Rob Pike, a Google engineer, best known for his work on Golang, describes concurrency as a “composition of independently executing tasks” and he emphasizes that concurrency really is about _structuring a program_. That means that a concurrent program handles multiple tasks being in progress at the same time but not necessarily being executed simultaneously. The work on all tasks may be interleaved in some arbitrary order, similarly to the objects used by a juggler.

Concurrency is not parallelism. It tries to break tasks down. Those tasks don’t necessarily have to execute at the same time, though. Its primary goal is _structure_, not _parallelism_.

# Parallelism — It’s about execution

_Parallelism_, often mistakenly used synonymously for _concurrency_, is about the simultaneous execution of multiple things. If concurrency is about structure, then parallelism is about the execution of multiple tasks. We can say that concurrency makes the use of parallelism easier, but it is not even a prerequisite since we can have [parallelism](https://en.wikipedia.org/wiki/Parallel_computing) without concurrency.

Conclusively, as Rob Pike describes it: “Concurrency is about dealing with lots of things at once. Parallelism is about doing lots of things at once”. You can watch his talk _Concurrency is not Parallelism_ on [YouTube](https://youtu.be/oV9rvDllKEg).

# Coroutines in terms of concurrency and parallelism

Coroutines are about _concurrency_ first of all. They provide great tools that let us break down tasks into various chunks which are not executed simultaneously by default.

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

Since we use `runBlocking` from the main thread, it runs on the very same one. The `async` builders do not specify a separate [CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/) or [CoroutineContext](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines.experimental/-coroutine-context/index.html) and therefore also inherently run on `main`.  
We have two tasks run on the same thread, and they finish after a 1-second delay. That is possible since `delay` only suspends the coroutine and does not block `main`. The example is, as correctly described, an example of concurrency, not utilizing parallelism. Next, let's change the functions to something that really takes its time and see what happens.

# Parallel Coroutines

Instead of just delaying the coroutines, we let the functions `doSomethingUseful` calculate the [next probable prime](https://docs.oracle.com/javase/7/docs/api/java/math/BigInteger.html#nextProbablePrime()) based on a randomly generated `BigInteger` which happens to be a fairly expensive task (since this calculation is based on a random it will not run in deterministic time):

```kotlin
fun doSomethingUsefulOne(): BigInteger {
    log.debug("in doSomethingUsefulOne")
    return BigInteger(1500, Random()).nextProbablePrime()
}
```

Note that the `suspend` keyword is not necessary anymore and would actually be misleading. The function does not make use of other suspending functions and blocks the calling thread for the needed time. Running the code results in the following logs:

```log
22:22:04.716 [main] DEBUG logger - in runBlocking  
22:22:04.749 [main] DEBUG logger - in doSomethingUsefulOne  
22:22:05.595 [main] DEBUG logger - Prime calculation took 844 ms  
22:22:05.602 [main] DEBUG logger - in doSomethingUsefulOne  
22:22:08.241 [main] DEBUG logger - Prime calculation took 2638 ms  
Completed in 3520 ms
```

As we can easily see, the tasks still run concurrently as in with `async` coroutines but don't execute at the same time anymore. The overall runtime is the sum of both sub-calculations (roughly). After changing the suspending code to blocking code, the result changes and we don't win any time while executing anymore.

# Note on the example

Let me note that I find the example provided in the documentation slightly misleading as it concludes with “This is twice as fast because we have concurrent execution of two coroutines” after applying `async` coroutine builders to the previously sequentially executed code. It only is "twice as fast" since the concurrently executed coroutines just delay in a non-blocking way. The example gives the impression that we get "parallelism" for free although it's only meant to demonstrate asynchronous programming, as far as I can tell.

Now how can we make coroutines run in parallel? To fix our prime example from above, we need to dispatch these tasks on some worker threads to not block the main thread anymore. We have a few possibilities to make this work.

# Making coroutines run in parallel

## 1. Run in [`GlobalScope`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/)

We can spawn a coroutine in the `GlobalScope`. That means that the coroutine is not bound to any `Job` and only limited by the lifetime of the whole application. That is the behavior we know from spawning new threads. It's hard to keep track of global coroutines, and the whole approach seems naive and error-prone. Nonetheless, running in this global scope dispatches a coroutine onto [Dispatchers.Default](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-default.html), a shared thread pool managed by the kotlinx.coroutines library. By default, the maximal number of threads used by this dispatcher is equal to the number of available CPU cores, but is at least two.

Applying this approach to our example is simple. Instead of running `async` in the scope of `runBlocking`, i.e., on the main thread, we spawn them in `GlobalScope`:

```kotlin
val time = measureTimeMillis {  
    val one = GlobalScope.async { doSomethingUsefulOne() }  
    val two = GlobalScope.async { doSomethingUsefulTwo() }  
}
```

The output verifies that we now run in roughly `max(time(calc1), time(calc2))`:

```log
22:42:19.375 [main] DEBUG logger - in runBlocking  
22:42:19.393 [DefaultDispatcher-worker-1] DEBUG logger - in doSomethingUsefulOne  
22:42:19.408 [DefaultDispatcher-worker-4] DEBUG logger - in doSomethingUsefulOne  
22:42:22.640 [DefaultDispatcher-worker-1] DEBUG logger - Prime calculation took 3245 ms  
22:42:23.330 [DefaultDispatcher-worker-4] DEBUG logger - Prime calculation took 3922 ms  
Completed in 3950 ms
```

We successfully applied parallelism to our concurrent example. As I said though, this fix is naive and can be improved further.

## 2. Specify a coroutine dispatcher

Instead of spawning `async` in the `GlobalScope`, we can still let them run in the scope of, i.e., as a child of, `runBlocking`. To get the same result, we explicitly set a coroutine dispatcher now:

```kotlin
val time = measureTimeMillis {  
    val one = async(Dispatchers.Default) { doSomethingUsefulOne() }  
    val two = async(Dispatchers.Default) { doSomethingUsefulTwo() }  
    println("The answer is ${one.await() + two.await()}")  
}
```

This adjustment leads to the same result as before while not losing the child-parent structure we want. We can still do better though. Wouldn’t it be most desirable to have real suspending functions again? Instead of taking care of not blocking the main thread while executing blocking functions, it would be best only to call suspending functions that don’t block the caller.

## 3. Make blocking function suspending

We can use [withContext](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-context.html) which "immediately applies dispatcher from the new context, shifting execution of the block into the different thread inside the block, and back when it completes":

```kotlin
suspend fun doSomethingUsefulOne(): BigInteger = withContext(Dispatchers.Default) {
    measureTimedValue {
        println("in doSomethingUsefulOne")
        BigInteger(1500, Random()).nextProbablePrime()
    }
}.also {
    println("Prime calculation took ${it.duration} ms")
}.value
```

With this approach, we confine the execution of dispatched tasks to the prime calculation inside the suspending function. The output nicely demonstrates that only the actual prime calculation happens on a different thread while everything else stays on main. When has multi-threading ever been that easy? I really like this solution the most.

```log
23:00:20.591 [main] DEBUG logger - in runBlocking  
23:00:20.648 [DefaultDispatcher-worker-1] DEBUG logger - in doSomethingUsefulOne  
23:00:20.714 [DefaultDispatcher-worker-2] DEBUG logger - in doSomethingUsefulOne  
23:00:21.132 [main] DEBUG logger - Prime calculation took 413 ms  
23:00:23.971 [main] DEBUG logger - Prime calculation took 3322 ms  
Completed in 3371 ms
```

# Caution: We use Concurrency and Parallelism interchangeably although we should not

As already mentioned in the introductory part of this article, we often use the terms parallelism and concurrency as synonyms of each other. I want to show you that even the Kotlin documentation does not clearly differentiate between both terms. The section on “Shared mutable state and concurrency” (as of 11/5/2018, may be changed in future) introduces with:

> Coroutines can be executed *concurrently* using a *multi-threaded dispatcher* like the `Dispatchers.Default`. It presents all the usual concurrency problems. The main problem being synchronization of access to shared mutable state. Some solutions to this problem in the land of coroutines are similar to the solutions in the multi-threaded world, but others are unique.

This sentence should really read “Coroutines can be executed in _parallel_ using multi-threaded dispatchers like `Dispatchers.Default`..."

# Conclusion

It’s important to know the difference between concurrency and parallelism. We learned that concurrency is mainly about dealing with many things at once while parallelism is about executing many things at once. Coroutines provide sophisticated tools to enable concurrency but don’t give us parallelism for free. In some situations, it will be necessary to dispatch blocking code onto some worker threads to let the main program flow continue. Please remember that we mostly need parallelism for CPU-intensive and performance-critical tasks. In most scenarios, it might be just fine to don’t worry about parallelism and be happy about the fantastic concurrency we get from coroutines.

----

## References
1. https://betterprogramming.pub/the-difference-between-concurrency-and-parallelism-explained-using-kotlin-83f4159581d