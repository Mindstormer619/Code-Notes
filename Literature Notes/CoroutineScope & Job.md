# CoroutineScope & Job
> Created: 2022-09-01 18:18

## CoroutineScope

An interface which defines a _scope_ for new coroutines. The scope contains a _coroutine context_ and several extension methods to build new coroutines (like `launch`, `async` etc.). An outer scope cannot complete until all its children coroutines complete.

You can construct an instance using `CoroutineScope()` or `MainScope()` factory functions, but you will need to cancel the coroutine scopes when no longer needed. You can also use `suspend coroutineScope {}` builder for constructing a new parent scope within an existing suspending function.



----

## References
1. [[Fleeting Notes/CoroutineScope & Job]]
2. https://kotlinlang.org/docs/coroutines-basics.html#structured-concurrency