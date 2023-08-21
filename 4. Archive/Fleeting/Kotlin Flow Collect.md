```kotlin
public suspend inline fun <T> Flow<T>.collect(
    crossinline action: suspend (T) → Unit
): Unit
```

Terminal flow operator that collects the given flow with a provided action. If any exception occurs during collect or in the provided flow, this exception is rethrown from this method.