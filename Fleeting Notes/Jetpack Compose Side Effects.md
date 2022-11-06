# Jetpack Compose Side Effects
> Created: 2022-01-12 22:25

## Effect APIs

### LaunchedEffect

When `LaunchedEffect` enters the Composition, it launches a coroutine with the block of code passed as a parameter. The coroutine will be cancelled if `LaunchedEffect` leaves the composition. If `LaunchedEffect` is recomposed with different keys (see the [Restarting Effects](https://developer.android.com/jetpack/compose/side-effects#restarting-effects) section below), the existing coroutine will be cancelled and the new suspend function will be launched in a new coroutine.

> What this means:
> 1. LaunchedEffect *will not run* if the conditions leading to it being declared get invalidated.
> 2. Any running LaunchedEffect under the above condition will get cancelled automatically (that's what the documentation means by "leaving the composition").
> 3. If the key on which the LaunchedEffect is created is changed, the effect is retriggered.

```kotlin
@Composable
fun MyScreen(
    state: UiState<List<Movie>>,
    scaffoldState: ScaffoldState = rememberScaffoldState()
) {

    // If the UI state contains an error, show snackbar
    if (state.hasError) {

        // `LaunchedEffect` will cancel and re-launch if
        // `scaffoldState.snackbarHostState` changes
        LaunchedEffect(scaffoldState.snackbarHostState) {
            // Show snackbar using a coroutine, when the coroutine is cancelled the
            // snackbar will automatically dismiss. This coroutine will cancel whenever
            // `state.hasError` is false, and only start when `state.hasError` is true
            // (due to the above if-check), or if `scaffoldState.snackbarHostState` changes.
            scaffoldState.snackbarHostState.showSnackbar(
                message = "Error message",
                actionLabel = "Retry message"
            )
        }
    }

    Scaffold(scaffoldState = scaffoldState) {
        /* ... */
    }
}
```

### rememberCoroutineScope

In order to launch a coroutine outside of a composable, but scoped so that it will be automatically canceled once it leaves the composition, use [`rememberCoroutineScope`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#rememberCoroutineScope(kotlin.Function0)). Also use `rememberCoroutineScope` whenever you need to control the lifecycle of one or more coroutines manually, for example, cancelling an animation when a user event happens.

> An example of when this is needed, is in event handlers. Event handlers' scope is **outside** the composable.

`rememberCoroutineScope` is a composable function that returns a `CoroutineScope` bound to the point of the Composition where it's called. The scope will be cancelled when the call leaves the Composition.

```kotlin
@Composable
fun MoviesScreen(scaffoldState: ScaffoldState = rememberScaffoldState()) {

    // Creates a CoroutineScope bound to the MoviesScreen's lifecycle
    val scope = rememberCoroutineScope()

    Scaffold(scaffoldState = scaffoldState) {
        Column {
            /* ... */
            Button(
                onClick = {
                    // Create a new coroutine in the event handler to show a snackbar
                    scope.launch {
                        scaffoldState.snackbarHostState.showSnackbar("Something happened!")
                    }
                }
            ) {
                Text("Press me")
            }
        }
    }
}
```

### rememberUpdatedState

The objective of the example below is to wait 5 seconds and then print out the latest value of `buttonColor` as of when the timer ends.

```kotlin
@Composable
fun Timer(
    buttonColor: String = "Unknown"
) {
    val timerDuration = 5000L
    println("Composing timer with colour : $buttonColor")
    val latestButtonColor by rememberUpdatedState(newValue = buttonColor)
    LaunchedEffect(key1 = Unit, block = {
        startTimer(timerDuration) {
            println("Timer ended")
            println("[1] Last pressed button color is $buttonColor") // "Unknown"
            println("[2] Last pressed button color is $latestButtonColor") // actual button color value after 5 seconds have passed
        }
    })
}
```

In the above example, if we do not use `rememberUpdatedState`, then the LaunchedEffect only accesses the *original* value of `buttonColor`, since it never relaunches even when `Timer` is recomposed. However, we also cannot use `buttonColor` as the `key1` for the LaunchedEffect because, if we did, LaunchedEffect would get *relaunched* every time the button color changed, which would cancel the existing timer and restart the entire timer all over again.

### DisposableEffect

Allows you to provide a "destructor method" to clean up the state of the effect when the effect leaves the composition, or if the `key` changes:

```kotlin
@Composable
fun HomeScreen(
    lifecycleOwner: LifecycleOwner = LocalLifecycleOwner.current,
    onStart: () -> Unit, // Send the 'started' analytics event
    onStop: () -> Unit // Send the 'stopped' analytics event
) {
    // Safely update the current lambdas when a new one is provided
    val currentOnStart by rememberUpdatedState(onStart)
    val currentOnStop by rememberUpdatedState(onStop)

    // If `lifecycleOwner` changes, dispose and reset the effect
    DisposableEffect(lifecycleOwner) {
        // Create an observer that triggers our remembered callbacks
        // for sending analytics events
        val observer = LifecycleEventObserver { _, event ->
            if (event == Lifecycle.Event.ON_START) {
                currentOnStart()
            } else if (event == Lifecycle.Event.ON_STOP) {
                currentOnStop()
            }
        }

        // Add the observer to the lifecycle
        lifecycleOwner.lifecycle.addObserver(observer)

        // When the effect leaves the Composition, remove the observer
        onDispose {
            lifecycleOwner.lifecycle.removeObserver(observer)
        }
    }

    /* Home screen content */
}

```

In the above, DisposableEffect calls `onDispose` when it leaves the composable, or if `lifecycleOwner` changes (in which case it is "destroyed" and then called again).

A `DisposableEffect` must include an `onDispose` clause as the final statement in its block of code. Otherwise, the IDE displays a build-time error.



----

## References
1. https://developer.android.com/jetpack/compose/side-effects
2. https://proandroiddev.com/jetpack-compose-side-effects-iii-rememberupdatedstate-c8df7b90a01d