# Java Performance - An Approach To Performance Testing
> Created: 2022-05-22 22:11
> #book 

## Approaches

### Microbenchmarks

Test designed to measure a very small unit of performance, e.g. a synchronized method vs an unsynchronized version, thread creation vs thread pool etc.

#### Understand what you're measuring

It is important to be careful with implementing microbenchmarks, because they might end up not really measuring what you think it measures. Consider the following for benchmarking a Fibonacci method:

```kotlin
fun main() {
	val start = System.currentTimeMillis()
	var r = 0
	repeat(500) {
		r = fibonacci(50)
	}
	val end = System.currentTimeMillis()
	println("Elapsed time: ${end - start}")
}


fun fibonacci(n: Int) {
	// elided: function that returns nth fibo number
}
```

The problem with the above is that it literally removes all the code between the `start` and `end` declaration. Why? The compiler notices that the result of `r` is unused, and therefore optimizes it away, given that `fibonacci` has no side effects.

To ensure the result is read, declare `r` as a `volatile` instance variable. [Kotlin volatile properties](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.jvm/-volatile/).

> Article on Kotlin's volatile from Baeldung: https://www.baeldung.com/kotlin/volatile-properties. Notes in [[Volatility in Kotlin]].

> **Threaded microbenchmarks.** Be very careful with these, and in most cases this isn't a good idea. Microbenchmarked code is focused on very specific portions, and in most cases you'll end up benchmarking the JVM's capabilities of handling synchronization rather than the part of the program you're actually trying to test. Most of the benchmarked code is going to spend its time within the synchronized section, and the chance of two threads "colliding" is artifically high.

#### Avoid extraneous operations

In the case above, you're looking for the performance of the `fibonacci` method, but this varies for different ranges of numbers (`fibo(1)` will run a different code path (edge case) as compared to `fibo(100)`). Therefore, you might want to run the test using random integers. If you write the portion under test this way:

```kotlin
repeat(1000) {
	r = fibonacci(Random.nextInt())
}
```

... you're also including the cost of calculating each random number. Ideally, you should precalculate the list and *then* begin your benchmark.

#### Measure for the correct input

The range of possible inputs (in real-world scenarios) must be considered when running microbenchmarks. You can't simply compare two implementations and state that one method is faster than another when the range of inputs doesn't represent real-world data.

For the above example, consider a method which checks for out-of-bounds inputs (like negative numbers) and throws the exception *before* going through the remaining calculations, versus a much faster general-case implementation that doesn't check for the negative number edge case. In case your test input range (generated randomly) happens to have loads of negative numbers, it's gonna say that the former implementation is way faster, even though it's actually slower for all *real* cases where this application is used (no one's gonna be looking for the -3'rd Fibonacci number).

#### Warmup

Java code is JIT compiled, which means you need to run some cycles as a "warmup" period before you actually microbenchmark your code. Otherwise you end up measuring the performance of the compilation rather than the actual code you wrote.

### Macrobenchmarks

> **Question: As a Java application engineer, why would you want to be concerned about testing the application with any active DB connections to real databases? Why not just mock them out? Aren't performance issues with the database supposed to be a DB engineer problem?**
> Even as an application engineer:
> + DB connection buffers take up memory space
> + Network calls can get saturated depending on the amount of data
> + CPU/cache pipeline optimization testing.

The idea of macrobenchmarks is to test the various modules of the system *in conjunction with* the other modules in the system -- we don't microfocus on one specific part of the system, rather we test the module without mocking out the rest of the system.

When testing in isolation, you may end up spending time on optimizing parts of the system that aren't the bottleneck (or at least, aren't the *sole* bottleneck when tested in tandem with the rest of the system). Even if they seem important to the business, it doesn't make your application faster.

> **Multiple JVM Testing**
> Consider the production environment which your application lives in when you're testing, especially if you're gonna run more than one JVM application at a time. Example: when GC cycles run, the JVM (default config) is set to run at 100% CPU usage on *all processors*. This means that your average CPU usage might be 40% which is actually 30% normal usage and 100% GC cycle usage -- which is fine when run in isolation, but will affect performance when multiple JVM instances are considered.

### Mesobenchmarks

This is really a question of semantics. If you view microbenchmarks as being performance tests of very specific small sections of the code, and macrobenchmarks as tests of the module in context of the entire system being active, then mesobenchmarks are essentially testing one subset of the code paths in isolation.

An example would be the time to fetch a single templated page from the server. This covers a significant amount of code -- network calls, templating logic, writing the response -- but isn't a macrobenchmark as it ignores the security layer (authentication), databases etc.

## Throughput, Batching, Response Time

### Batch Measurements

Very simple -- the time the application takes in total to accomplish a certain task. For instance, calculating the stock price for 10000 stocks in a single execution.

**Application warm-up:** Java bytecode is JIT-compiled, which means that it speeds up certain frequently used code paths as it executes. This means that for true performance benchmarks on long-running services / batch architectures, you need to account for the warmup time by letting the application run for a while before profiling. Also consider data caching which might be happening at different layers of your app while taking warmup into account.

### Throughput

The amount of work that can be accomplished in a certain period of time. This is generally measured as transactions (TPS), operations (OPS) or requests (RPS) per second, taken as an aggregated result across multiple client threads.

Consider the risk that the client might hit a performance wall and not be able to send requests quickly enough to the server. In this case, you'll end up measuring the performance of the client, not the server, and the metrics will be incorrect.

Average response time measurements accompany throughput tests. A 400 OPS 0.3ms response time app is generally *worse* than a 500OPS 0.5ms response time app.

Usually measured post-warmup cycles.

### Response Time

The amount of time between sending a request from the client and receiving the response from the server. Also termed **latency**.

Clients in a response time test generally sleep a short while between requests. This is referred to as **think time** -- throughput tests generally have ZERO think time (for obvious reasons). For response tests, you're trying to mimic more closely what a user does -- waiting between clicks and interactions.

Measuring response time:
1. As an average across all requests
2. Percentile measurements -- x% of requests had a response time below y ms.

**Percentile Measurements**

90th% response time = 1.5s means that 90% of your responses arrived within 1.5s, and the remaining 10% took longer than that.

Outliers heavily affect the average response time. Pay attention to outlier spikes if you want to reduce the average response time with less effort.

Focusing on percentile measures is generally better because it benefits the majority of users -- but also microfocusing on that and ignoring average response times can lead to cases where you miss massive outliers which could be symptoms of a bigger problem.

> Load testing library: http://faban.org/

## Variability

Test results vary over time. Even when you run a performance test multiple times and take the average, you cannot be completely certain that the average reflects "real" performance improvements, or whether it's a result of random factors (i.e. other processes happened to not be consuming resources on the machine, network happened to be uncongested etc.).

| Iterations | Baseline | Specimen |
| ---------- | -------- | -------- |
| 1          | 1.0s     | 0.5s     |
| 2          | 0.8s     | 1.25s    |
| 3          | 1.2s     | 0.5s     |
| Average    | 1s       | 0.75s    |

In the above example, it seems that the specimen in the regression test has achieved a 25% improvement. However, it is 43% probable that the specimen and the baseline have the same performance. 57% of the time, the performance is *not* the same — this does not mean that the performance is 25% better, or even that it is better at all. It just means there's a 57% probability that the specimen and the baseline are *different* in performance, i.e. we don't have a **null hypothesis**.

> To calculate probability values like the one above (**p-value**), we use [Student's *t*-test](https://en.wikipedia.org/wiki/Student%27s_t-test).

The way you phrase this result is as follows: there is a 57% probability that the performance of the specimen differs from the baseline, and the best estimate of that difference is 25%.

You want the p-value to be smaller than the α-value, which is a measure of the acceptable fraction of the sample and the baseline being the same. Typical α-values are 0.1 (10%), 0.5 (5%) or 0.01 (1%) -- in this case, the p-value of 0.43 is super high and therefore indicative of an improvement that is **not statistically significant**.

## Test Early, Test Often

In general, you want to test as early and as often as possible, even if you plan to fix the regressions or performance issues only later on. 

Performance testing is a little tricky to test early in the lifecycle because you don't actually have the final code / requirements which are gonna be a part of your full application -- and as you write more code it's going to affect the overall performance of the app. This can be at odds with other forms of testing (like unit tests for functionality, or security tests) where typically you fix the issues as soon as they are discovered.

Follow these guidelines:

+ **Automate everything.** All performance tests should be scripted. All the way from deploying new code, running multiple tests, running t-test analysis, confidence level reports and measuring differences.
+ **Measure everything.** The automation must gather every conceivable piece of measurement from the test run. CPU/memory/storage/network usage, app logs, GC logs, heap dumps etc. Throw away data later when you're done debugging performance issues and don't need it anymore, but in that moment it'll guide the analysis of any debugging that you need to do. The more data that's available, the faster it is to track down clues.
+ **Run on the target system.** Actually run your performance tests in hardware/software that is as close to the actual production system as possible.

----

## References
1. [Java Performance: The Definitive Guide](https://www.oreilly.com/library/view/java-performance-the/9781449363512/) Chapter 2