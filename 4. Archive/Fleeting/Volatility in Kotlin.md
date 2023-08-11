# Volatility in Kotlin
> Created: 2022-05-22 22:26

```kotlin
object TaskExecutor : Runnable {
    private var shouldContinue = true

    override fun run() {
        while (shouldContinue) {}
        println("Done")
    }

    fun stop() {
        shouldContinue = false
    }
}
```

The _TaskExecutor_ singleton object will continue the spin wait as long as the _shouldContinue_ variable is set to _true_:

```kotlin
fun main() {
    val thread = Thread(TaskExecutor)
    thread.start()

    TaskExecutor.stop()

    thread.join()
}
```

One might expect that once we call the _stop()_ method, the busy-wait will end immediately and the _TaskExecutor_ will print _"Done"_ to the console.

**However, due to the nature of [shared multiprocessor architectures](https://www.baeldung.com/java-volatile#shared-multiprocessor-architecture) and CPU caches, the busy-wait might end with a delay. Adding insult to injury, it might not end at all**.

To avoid these sorts of processor or runtime optimizations on the _shouldContinue_ variable, we should annotate that property with a _[@Volatile](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.jvm/-volatile/)_ annotation:

```kotlin
object TaskExecutor : Runnable {
    @Volatile private var shouldContinue = true
    // same as before
}
```

The _@Volatile_ annotation will mark the JVM backing field of the annotated property as _volatile_. **Thus, the writes to this field are immediately made visible to other threads**. Moreover, the reads will always see the latest changes.

Simply put, the _@Volatile_ annotation is the equivalent of Java’s _volatile_ keyword in Kotlin.

## JVM Representation

```bash
$ kotlinc TaskExecutor.kt
$ javap -v -p -c TaskExecutor
# omitted for brevity
private static volatile boolean shouldContinue;
    descriptor: Z
    flags: (0x004a) ACC_PRIVATE, ACC_STATIC, ACC_VOLATILE
```

----

## References
1. https://www.baeldung.com/kotlin/volatile-properties
2. https://www.baeldung.com/java-volatile