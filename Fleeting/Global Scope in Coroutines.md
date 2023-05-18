# Global Scope in Coroutines
> Created: 2022-08-02 23:02

A global [CoroutineScope](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html) not bound to any job. Global scope is used to launch top-level coroutines which are operating on the whole application lifetime and are not cancelled prematurely.

Active coroutines launched in `GlobalScope` do not keep the process alive. They are like daemon threads.

This is a **delicate** API. It is easy to accidentally create resource or memory leaks when `GlobalScope` is used. A coroutine launched in `GlobalScope` is not subject to the principle of structured concurrency, so if it hangs or gets delayed due to a problem (e.g. due to a slow network), it will stay working and consuming resources. For example, consider the following code:

```kotlin
fun loadConfiguration() {
    GlobalScope.launch {
        val config = fetchConfigFromServer() // network request
        updateConfiguration(config)
    }
}
```

A call to `loadConfiguration` creates a coroutine in the `GlobalScope` that works in background without any provision to cancel it or to wait for its completion. If a network is slow, it keeps waiting in background, consuming resources. Repeated calls to `loadConfiguration` will consume more and more resources.

### Possible replacements

In many cases uses of `GlobalScope` should be removed, marking the containing operation with `suspend`, for example:

```kotlin
suspend fun loadConfiguration() {
    val config = fetchConfigFromServer() // network request
    updateConfiguration(config)
}
```

In cases when `GlobalScope.launch` was used to launch multiple concurrent operations, the corresponding operations shall be grouped with [coroutineScope](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html) instead:

```kotlin
// concurrently load configuration and data
suspend fun loadConfigurationAndData() {
    coroutineScope {
        launch { loadConfiguration() }
        launch { loadData() }
    }
}
```

### Legitimate use-cases

There are limited circumstances under which `GlobalScope` can be legitimately and safely used, such as top-level background processes that must stay active for the whole duration of the application's lifetime. Because of that, any use of `GlobalScope` requires an explicit opt-in with `@OptIn(DelicateCoroutinesApi::class)`, like this:

```kotlin
// A global coroutine to log statistics every second, must be always active
@OptIn(DelicateCoroutinesApi::class)
val globalScopeReporter = GlobalScope.launch {
    while (true) {
        delay(1000)
        logStatistics()
    }
}
```

----

## References
1. https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/