# kotlin.concurrent.thread
> Created: 2022-05-07 00:34

```kotlin
fun thread(
    start: Boolean = true,
    isDaemon: Boolean = false,
    contextClassLoader: ClassLoader? = null,
    name: String? = null,
    priority: Int = -1,
    block: () -> Unit
): Thread
```

Utility method for creating a new thread easily, with a runnable `block` lambda.

----

## References
1. https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.concurrent/thread.html