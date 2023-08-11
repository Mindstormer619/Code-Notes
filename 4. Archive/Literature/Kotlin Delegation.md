# Kotlin Delegation
> Created: 2022-01-08 11:40

The [Delegate Pattern](https://en.wikipedia.org/wiki/Delegation_pattern) is used to handle inheritance (implementing an interface, for instance) via composition, by passing in an object that inherits the common base class and *delegating* all the method calls etc. to the held object.

In Kotlin, the delegate pattern can be implemented with minimal boilerplate via the keyword `by`.

As an example, we have a `Derived` class implement a `Base` interface by delegating to a held instance of `Base`:

```kotlin
interface Base {
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override fun print() { print(x) }
}

class Derived(b: Base) : Base by b

fun main() {
    val b = BaseImpl(10)
    Derived(b).print() // 10
}
```



----

## References
1. [[4. Archive/Fleeting/Kotlin Delegation]]