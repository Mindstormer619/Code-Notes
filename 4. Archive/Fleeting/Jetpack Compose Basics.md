# Jetpack Compose Basics
> Created: 2022-01-08 23:03

## Declarative Programming
The technique works by *conceptually* regenerating the entire screen from scratch, then applying only the necessary changes. This approach avoids the complexity of manually updating a stateful view hierarchy. Compose is a declarative UI framework.

Basically you *declare* the UI in code, and updates are taken care of automatically. One challenge with regenerating the entire screen is that it is potentially expensive, in terms of time, computing power, and battery usage. To mitigate this cost, Compose intelligently chooses which parts of the UI need to be redrawn at any given time. This does have some implications for how you design your UI components.

## Composable Functions
+ The function is annotated with the `@Composable` annotation.
+ The function doesn't return anything.
+ This function is fast, [idempotent](https://en.wikipedia.org/wiki/Idempotence#Computer_science_meaning), and free of _side-effects_.

## Recomposition Tips

**Never** depend on side-effects from executing composable functions, since a function's recomposition may be skipped. If you do, users may experience strange and unpredictable behavior in your app.

Composable functions might be re-executed as often as every frame, such as when an animation is being rendered. Composable functions should be fast to avoid jank during animations. If you need to do expensive operations, such as reading from shared preferences, do it in a background coroutine and pass the value result to the composable function as a parameter.

+ Composable functions can execute in any order.
+ Composable functions can execute in parallel.
+ Recomposition skips as many composable functions and lambdas as possible.
+ Recomposition is optimistic and may be canceled.
+ A composable function might be run quite frequently, as often as every frame of an animation.

### Execution Order

For example, suppose you have code like this to draw three screens in a tab layout:

```kotlin
@Composable
fun ButtonRow() {
	MyFancyNavigation {
		StartScreen()
		MiddleScreen()
		EndScreen()
	}
}
```

The calls to `StartScreen`, `MiddleScreen`, and `EndScreen` might happen in any order. This means you can't, for example, have `StartScreen()` set some global variable (a side-effect) and have `MiddleScreen()` take advantage of that change. Instead, each of those functions needs to be self-contained.

### Avoid Side Effects

```kotlin
@Composable
@Deprecated("Example with bug")
fun ListWithBug(myList: List<String>) {
    var items = 0

    Row(horizontalArrangement = Arrangement.SpaceBetween) {
        Column {
            for (item in myList) {
                Text("Item: $item")
                items++ // Avoid! Side-effect of the column recomposing.
            }
        }
        Text("Count: $items")
    }
}
```

----

## State

Composable functions can store a single object in memory by using the [`remember`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#remember(kotlin.Function0)) composable. A value computed by `remember` is stored in the Composition during initial composition, and the stored value is returned during recomposition. `remember` can be used to store both mutable and immutable objects.

**The `by` delegate syntax requires the following imports:**
```kotlin
import androidx.compose.runtime.getValue
import androidx.compose.runtime.setValue
```
For some dumb reason Compose does not define these as methods on the class, but rather as extensions.

Jetpack Compose doesn't require that you use `MutableState<T>` to hold state. Jetpack Compose supports other observable types. Before reading another observable type in Jetpack Compose, you must convert it to a `State<T>` so that Jetpack Compose can automatically recompose when the state changes.

### State Hoisting

State hoisting in Compose is a pattern of moving state to a composable's caller to make a composable stateless. The general pattern for state hoisting in Jetpack Compose is to replace the state variable with two parameters:

+ **`value: T`:** the current value to display
+ **`onValueChange: (T) -> Unit`:** an event that requests the value to change, where `T` is the proposed new value

```kotlin
@Composable
fun HelloScreen() {
    var name by rememberSaveable { mutableStateOf("") }

    HelloContent(name = name, onNameChange = { name = it })
}

@Composable
fun HelloContent(name: String, onNameChange: (String) -> Unit) {
    Column(modifier = Modifier.padding(16.dp)) {
        Text(
            text = "Hello, $name",
            modifier = Modifier.padding(bottom = 8.dp),
            style = MaterialTheme.typography.h5
        )
        OutlinedTextField(
            value = name,
            onValueChange = onNameChange,
            label = { Text("Name") }
        )
    }
}
```

In general, you want state to be present at the *lowest common ancestor* where the particular piece of state is used, in the UI hierarchy (in the above example it probably might have helped to leave it in `HelloContent` considering right now there's no other Composable that uses it).

> **Key Point:** When hoisting state, there are three rules to help you figure out where state should go:
> 
> 1.  State should be hoisted to at _least_ the **lowest common parent** of all composables that use the state (read).
> 2.  State should be hoisted to at _least_ the **highest level it may be changed** (write).
> 3.  If **two states change in response to the same events** they should be **hoisted together.**
> 
> You can hoist state higher than these rules require, but underhoisting state will make it difficult or impossible to follow unidirectional data flow.

Unidirectional data flow:
![[Pasted image 20220112223352.png]]

State is passed down to the child composables, while events bubble up to the parent level where the state is modified based on the event.

### State Holders

![State holder dependencies](https://developer.android.com/images/jetpack/compose/state-dependencies.svg)

```kotlin
// Plain class that manages App's UI logic and UI elements' state
class MyAppState(
  val scaffoldState: ScaffoldState,
  val navController: NavHostController,
  private val resources: Resources,
  /* ... */
) {
  val bottomBarTabs = /* State */

  // Logic to decide when to show the bottom bar
  val shouldShowBottomBar: Boolean
    get() = /* ... */

  // Navigation logic, which is a type of UI logic
  fun navigateToBottomBarRoute(route: String) { /* ... */ }

  // Show snackbar using Resources
  fun showSnackbar(message: String) { /* ... */ }
}

@Composable
fun rememberMyAppState(
  scaffoldState: ScaffoldState = rememberScaffoldState(),
  navController: NavHostController = rememberNavController(),
  resources: Resources = LocalContext.current.resources,
  /* ... */
) = remember(scaffoldState, navController, resources, /* ... */) {
  MyAppState(scaffoldState, navController, resources, /* ... */)
}

// --------

@Composable
fun MyApp() {
  MyTheme {
    val myAppState = rememberMyAppState()
    Scaffold(
      scaffoldState = myAppState.scaffoldState,
      bottomBar = {
        if (myAppState.shouldShowBottomBar) {
          BottomBar(
            tabs = myAppState.bottomBarTabs,
            navigateToRoute = {
              myAppState.navigateToBottomBarRoute(it)
            }
          )
        }
      }
    ) {
      NavHost(navController = myAppState.navController, "initial") { /* ... */ }
    }
  }
}
```



----

## References
1. https://youtu.be/SMOhl9RK0BA
2. https://developer.android.com/jetpack/compose/mental-model
3. https://developer.android.com/jetpack/compose/state
4. https://youtu.be/rmv2ug-wW4U