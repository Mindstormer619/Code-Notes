# Kotlin Delegation
> Created: 2022-01-07 23:05

## Type Delegation
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
    Derived(b).print()
}
```

The `by`-clause in the supertype list for `Derived` indicates that `b` will be stored internally in objects of `Derived` and the compiler will generate all the methods of `Base` that forward to `b`.

Members overridden in this way do not get called from the members of the delegate object, which can only access its own implementations of the interface members.

## Property Delegation
```kotlin
class Example {
    var p: String by Delegate()
}
```

```kotlin
import kotlin.reflect.KProperty

class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name}' in $thisRef.")
    }
}
```

When you read from `p`, which delegates to an instance of `Delegate`, the `getValue()` function from `Delegate` is called. Its first parameter is **the object you read `p` from**, and the second parameter holds a description of `p` itself (for example, you can take its name).

### Standard Delegates
#### lazy
```kotlin
val lazyValue: String by lazy {
    println("computed!")
    "Hello"
}

fun main() {
    println(lazyValue)
    println(lazyValue)
}
```

By default, the evaluation of lazy properties is _synchronized_: the value is computed only in one thread, but all threads will see the same value. If the synchronization of the initialization delegate is not required to allow multiple threads to execute it simultaneously, pass `LazyThreadSafetyMode.PUBLICATION` as a parameter to `lazy()`.

If you're sure that the initialization will always happen in the same thread as the one where you use the property, you can use `LazyThreadSafetyMode.NONE`. It doesn't incur any thread-safety guarantees and related overhead.

#### Observable
```kotlin
import kotlin.properties.Delegates

class User {
    var name: String by Delegates.observable("<no name>") {
        prop, old, new -> println("$old -> $new")
    }
}

fun main() {
    val user = User()
    user.name = "first"
    user.name = "second"
}
```

#### Vetoable
```kotlin
var max: Int by Delegates.vetoable(0) { property, oldValue, newValue ->
    newValue > oldValue
}

println(max) // 0

max = 10
println(max) // 10

max = 5
println(max) // 10
```

### Delegating to a map
One common use case is storing the values of properties in a map. This comes up often in applications for things like parsing JSON or performing other dynamic tasks. In this case, you can use the map instance itself as the delegate for a delegated property.

```kotlin
class User(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int     by map
}
```

In this example, the constructor takes a map:

```kotlin
val user = User(mapOf(
    "name" to "John Doe",
    "age"  to 25
))
```

This also works for `var` ’s properties if you use a `MutableMap` instead of a read-only `Map`.


----

## References
1. https://kotlinlang.org/docs/delegation.html
2. https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-property/
3. https://kotlinlang.org/docs/delegated-properties.html
4. https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.properties/-delegates/vetoable.html