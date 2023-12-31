Compose regenerate entire screen on UI-elements change and is updating view correctly, depending on relevant state, avoiding unexpectency. Compose use intelligent logic which redraws only the views which are need to be redrawn.

`@Preview` annotation is used on composable functions with no parameters:
```kotlin 
@Composable  
fun MessageCard(name: String) {    Text(text = "Hello $name!")  
}  
  
@Preview  
@Composable  
fun PreviewMessageCard() {    
	MessageCard("Android")  
}
```
![[Pasted image 20231229171325.png]]
By default elements are placed on top of each other:
```kotlin
@Composable  
fun MessageCard(msg: Message) {    
	Text(text = msg.author)    
	Text(text = msg.body)  
}  
  
@Preview  
@Composable  
fun PreviewMessageCard() {    
	MessageCard(msg = Message("Lexi", "Hey, take a look at Jetpack Compose, it's great!")) 
}
```
![[Pasted image 20231229171618.png]]
Surface composable is a Material3 container, which allows to easily add design-elements to item. The Surface is responsible for:
- Clipping: Surface clips its children to the shape specified by shape
- Elevation: Surface draws a shadow to represent depth, where elevation represents the depth of this surface. If the passed shape is concave the shadow will not be drawn on Android versions less than 10.
- Borders: If shape has a border, then it will also be drawn.
- Background: Surface fills the shape specified by shape with the color. If color is Colors.surface, the ElevationOverlay from LocalElevationOverlay will be used to apply an overlay - by default this will only occur in dark theme. The color of the overlay depends on the elevation of this Surface, and the LocalAbsoluteElevation set by any parent surfaces. This ensures that a Surface never appears to have a lower elevation overlay than its ancestors, by summing the elevation of all previous Surfaces.
- Content color: Surface uses contentColor to specify a preferred color for the content of this surface - this is used by the Text and Icon components as a default color.
- Blocking touch propagation behind the surface.

Never depend on side-effects from executing composable functions, since a function's recomposition may be skipped. If you do, users may experience strange and unpredictable behavior in your app. A side-effect is any change that is visible to the rest of your app. For example, these actions are all dangerous side-effects:

- Writing to a property of a shared object
- Updating an observable in `ViewModel`
- Updating shared preferences
**Key Point:** The lifecycle of a composable is defined by the following events: entering the Composition, getting recomposed 0 or more times, and leaving the Composition.

![Diagram showing the lifecycle of a composable](https://developer.android.com/static/images/jetpack/compose/lifecycle-composition.png)

Recomposition is typically triggered by a change to a [`State<T>`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State) object. Compose tracks these and runs all composables in the Composition that read that particular `State<T>`, and any composables that they call that cannot be [skipped](https://developer.android.com/jetpack/compose/lifecycle#skipping).

```kotlin
@Composable  
fun LoginScreen(showError: Boolean) {    
	if (showError) {
		LoginError()
		}    
	LoginInput() // This call site affects where LoginInput is placed in Composition  
}  
  
@Composable  
fun LoginInput() { /* ... */ }  
  
@Composable  
fun LoginError() { /* ... */ }
```



In the code snippet above, `LoginScreen` will conditionally call the `LoginError` composable and will always call the `LoginInput` composable. Each call has a unique call site and source position, which the compiler will use to uniquely identify it.

![Diagram showing how the preceding code is recomposed if the showError flag is changed to true. The LoginError composable is added, but the other composables are not recomposed.](https://developer.android.com/static/images/jetpack/compose/lifecycle-showerror.png)

**Figure 3.** Representation of `LoginScreen` in the Composition when the state changes and a recomposition occurs. Same color means it hasn't been recomposed.

Even though `LoginInput` went from being called first to being called second, the `LoginInput` instance will be preserved across recompositions. Additionally, because `LoginInput` doesn’t have any parameters that have changed across recomposition, the call to `LoginInput` will be skipped by Compose.

**Key Point:** Use the `key` composable to help Compose identify composable instances in Composition. It's important when multiple composables are called from the same call site and contain side-effects or internal state.
```kotlin
@Composable  
fun MoviesScreenWithKey(movies: List<Movie>) {    
	Column {
		for (movie in movies) {          
		  key(movie.id) { // Unique ID for this movie
			  MovieOverview(movie)
		  }
	  }
  }  
}
```

Some composables have built-in support for the `key` composable. For example, `LazyColumn` accepts specifying a custom `key` in the `items` DSL.

```kotlin
@Composable
fun MoviesScreenLazy(movies: List<Movie>) {
    LazyColumn {
        items(movies, key = { movie -> movie.id }) { movie ->
            MovieOverview(movie)
        }
    }
}
```

Compose may skip recomposition, if all parameter values are stable and have not been changed. 
Stable parameters are:
- The result of `equals` for two instances will _forever_ be the same for the same two instances
- Object notifies compose if public parameter have changed
- Public properties are stable
- All primitives
- Strings
- Lambdas
- MutableState(from compose lib)
Compose considers a type stable only if it can prove it. For example, an interface is generally treated as not stable, and types with mutable public properties whose implementation could be immutable are not stable either.

If Compose is not able to infer that a type is stable, but you want to force Compose to treat it as stable, mark it with the [`@Stable`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/Stable) annotation.

```kotlin
// Marking the type as stable to favor skipping and smart recompositions.
@Stable
interface UiState<T : Result<T>> {
    val value: T?
    val exception: Throwable?

    val hasError: Boolean
        get() = exception != null
}
```

### Side effects
Side effect is a logic that changes the state of application outside of Compose function. Ideally compose functions should be side-effect free, but there are some situations in which application require some kind of side-effects. For example, showing one-time ui elements like snackbar or toast or navigating to another screen. These kinds of actions should be performed in a controllable manner. 
Compose provides different side-effect handlers for different situations
#### LaunchedEffect
LaunchedEffect is starting coroutine inside composable function to perform suspend functions. It cancells coroutine when LaunchedEffect leaves composition (if the root composable function is not in the UI hierarchy anymore) and also cancells coroutine if it is recomposed
```kotlin
@Composable
fun MyScreen(
    state: UiState<List<Movie>>,
    snackbarHostState: SnackbarHostState
) {

    // If the UI state contains an error, show snackbar
    if (state.hasError) {

        // `LaunchedEffect` will cancel and re-launch if
        // `scaffoldState.snackbarHostState` changes
        LaunchedEffect(snackbarHostState) {
            // Show snackbar using a coroutine, when the coroutine is cancelled the
            // snackbar will automatically dismiss. This coroutine will cancel whenever
            // `state.hasError` is false, and only start when `state.hasError` is true
            // (due to the above if-check), or if `scaffoldState.snackbarHostState` changes.
            snackbarHostState.showSnackbar(
                message = "Error message",
                actionLabel = "Retry message"
            )
        }
    }

    Scaffold(
        snackbarHost = {
            SnackbarHost(hostState = snackbarHostState)
        }
    ) { contentPadding ->
        // ...
    }
}
```

#### rememberCoroutineScope
rememberCoroutineScope is used to perform suspend function inside coroutine scope bound to composable function lifecycle in non-composable environment(like callback method).
```kotlin
@Composable
fun MoviesScreen(snackbarHostState: SnackbarHostState) {

    // Creates a CoroutineScope bound to the MoviesScreen's lifecycle
    val scope = rememberCoroutineScope()

    Scaffold(
        snackbarHost = {
            SnackbarHost(hostState = snackbarHostState)
        }
    ) { contentPadding ->
        Column(Modifier.padding(contentPadding)) {
            Button(
                onClick = {
                    // Create a new coroutine in the event handler to show a snackbar
                    scope.launch {
                        snackbarHostState.showSnackbar("Something happened!")
                    }
                }
            ) {
                Text("Press me")
            }
        }
    }
}
```

#### rememberUpdatedState
rememberUpdatedState is used to reference a value that should not impact side effect restart. It is used for example in cases where there is long-running operation and you want to get sure that in the end of it relevant data will be used.
```kotlin
@Composable
fun LandingScreen(onTimeout: () -> Unit) {

    // This will always refer to the latest onTimeout function that
    // LandingScreen was recomposed with
    val currentOnTimeout by rememberUpdatedState(onTimeout)

    // Create an effect that matches the lifecycle of LandingScreen.
    // If LandingScreen recomposes, the delay shouldn't start again.
    LaunchedEffect(true) {
        delay(SplashWaitTimeMillis)
        currentOnTimeout()
    }

    /* Landing screen content */
}

```

#### DisposableEffect
DisposableEffect is used in situations where there is a need to clean up resources before leaving side effect. For example registering observers:
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

#### SideEffect
SideEffect is used to share compose state to non-compose environment. Which means to send current data of composable to some third party logic. SideEffect guarantees to perform every time on successful recomposition:
```kotlin
@Composable
fun rememberFirebaseAnalytics(user: User): FirebaseAnalytics {
    val analytics: FirebaseAnalytics = remember {
        FirebaseAnalytics()
    }

    // On every successful composition, update FirebaseAnalytics with
    // the userType from the current User, ensuring that future analytics
    // events have this metadata attached
    SideEffect {
        analytics.setUserProperty("userType", user.userType)
    }
    return analytics
}

```

#### produceState
produceState is used to convert non-compose state to compose state. For example to convert some network request to trigger recomposition on response provided.  It launches the coroutine but is not obligated to use suspend functions. You can for example convert RxJava methods.
The following example shows how to use `produceState` to load an image from the network. The `loadNetworkImage` composable function returns a `State` that can be used in other composables.
```kotlin
@Composable
fun loadNetworkImage(
    url: String,
    imageRepository: ImageRepository = ImageRepository()
): State<Result<Image>> {

    // Creates a State<T> with Result.Loading as initial value
    // If either `url` or `imageRepository` changes, the running producer
    // will cancel and will be re-launched with the new inputs.
    return produceState<Result<Image>>(initialValue = Result.Loading, url, imageRepository) {

        // In a coroutine, can make suspend calls
        val image = imageRepository.load(url)

        // Update State with either an Error or Success result.
        // This will trigger a recomposition where this State is read
        value = if (image == null) {
            Result.Error
        } else {
            Result.Success(image)
        }
    }
}
```

#### derivedStateOf
derivedStateOf is used to avoid unnecessary recompositions. It allows to set the condition for when to perform recomposition. It is used when composable inputs is changing more often than you want recomposition to be performed:
```kotlin
@Composable
// When the messages parameter changes, the MessageList
// composable recomposes. derivedStateOf does not
// affect this recomposition.
fun MessageList(messages: List<Message>) {
    Box {
        val listState = rememberLazyListState()

        LazyColumn(state = listState) {
            // ...
        }

        // Show the button if the first visible item is past
        // the first item. We use a remembered derived state to
        // minimize unnecessary compositions
        val showButton by remember {
            derivedStateOf {
                listState.firstVisibleItemIndex > 0
            }
        }

        AnimatedVisibility(visible = showButton) {
            ScrollToTopButton()
        }
    }
}
```

#### snapshotFlow
snapshotFlow is used to convert compose state to cold Flow, which is than can be collected and used in another part of application.
The following example shows a side effect that records when the user scrolls past the first item in a list to analytics:
```kotlin
val listState = rememberLazyListState()

LazyColumn(state = listState) {
    // ...
}

LaunchedEffect(listState) {
    snapshotFlow { listState.firstVisibleItemIndex }
        .map { index -> index > 0 }
        .distinctUntilChanged()
        .filter { it == true }
        .collect {
            MyAnalyticsService.sendScrolledPastFirstItemEvent()
        }
}
```

#### Restarting effects
It is important to choose right parameter for side effect to trigger their recomposition right amount of times.
As a rule of thumb, mutable and immutable variables used in the effect block of code should be added as parameters to the effect composable. Apart from those, more parameters can be added to force restarting the effect. If the change of a variable shouldn't cause the effect to restart, the variable should be wrapped in [`rememberUpdatedState`](https://developer.android.com/jetpack/compose/side-effects#rememberupdatedstate). If the variable never changes because it's wrapped in a `remember` with no keys, you don't need to pass the variable as a key to the effect.

You can use a constant like `true` as an effect key to make it **follow the lifecycle of the call site**. There are valid use cases for it, like the `LaunchedEffect` example shown above. However, before doing that, think twice and make sure that's what you need. When you use constants for side effects, they will be triggered only once on composition enter.

### Compose phases
There are 3 phases of rendering compose layout:
1. Composition. During this phase Compose gather all information provided for ui and creates UI-tree
2. Layout. During this phase Compose measure all elements and children and places them in 2D screen.
3. Drawing. Compose uses all information about measurements and positions from the previous step and draws element on Canvas.
These steps happen every frame, but Compose is optimized well to avoid unneccesary work and it can skip steps if UI doesn't change.

### Compose state
To store state across recompositions of Composable functions you may use remember API. It allow to save data during Composition and clean it after Composable function is out of composition. remember stores value and MutableState provides a way to begin recomposition of functions that read from the corresponding value.
```kotlin
@Composable
fun HelloContent() {
    Column(modifier = Modifier.padding(16.dp)) {
        var name by remember { mutableStateOf("") }
        if (name.isNotEmpty()) {
            Text(
                text = "Hello, $name!",
                modifier = Modifier.padding(bottom = 8.dp),
                style = MaterialTheme.typography.bodyMedium
            )
        }
        OutlinedTextField(
            value = name,
            onValueChange = { name = it },
            label = { Text("Name") }
        )
    }
}
```
remember helps to save value during recompositions, but not during configuration changes. To handle it use rememberSaveable to put data to Bundle.
remember API provides methods to convert popular Android observable types to state. In order to use other observable types and trigger recomposition, you have to convert this type to MutableState<> inside Composable function using fitting extension functions.

### State hoisting
In order to develop more robust and flexible Composable functions you may consider making them stateless. Stateful Composables are the ones that store some state and therefore they are less testable and less reusable. The general way to make them more flexible is to move the state out of Composable to the caller function using:
	value: T,
	onValueChanged: (T) -> Unit
as Composable function parameters.
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
            style = MaterialTheme.typography.bodyMedium
        )
        OutlinedTextField(value = name, onValueChange = onNameChange, label = { Text("Name") })
    }
}
```

**Key Point:** When hoisting state, there are three rules to help you figure out where state should go:

1. State should be hoisted to at _least_ the **lowest common parent** of all composables that use the state (read).
2. State should be hoisted to at _least_ the **highest level it may be changed** (write).
3. If **two states change in response to the same events** they should be **hoisted together.**

You can hoist state higher than these rules require, but underhoisting state makes it difficult or impossible to follow unidirectional data flow.

If the amount of states that you need to keep track of is growing you may consider to use state holder classes
**Key Term:** **State holders** manage logic and state of composables.

Note that in other materials, state holders are also called _hoisted state objects_.
There are already implemented state holder for different built-in compose components.

We can use remember API to store expensive operations results to not repeatedly perform them after recomposition.

```kotlin
val brush = remember {
    ShaderBrush(
        BitmapShader(
            ImageBitmap.imageResource(res, avatarRes).asAndroidBitmap(),
            Shader.TileMode.REPEAT,
            Shader.TileMode.REPEAT
        )
    )
}
```
To invalidate value cached by remember, we can use key parameter which remember takes in. When this key changes, value is invalidated and recalculated again.
```kotlin
@Composable
private fun BackgroundBanner(
    @DrawableRes avatarRes: Int,
    modifier: Modifier = Modifier,
    res: Resources = LocalContext.current.resources
) {
    val brush = remember(key1 = avatarRes) {
        ShaderBrush(
            BitmapShader(
                ImageBitmap.imageResource(res, avatarRes).asAndroidBitmap(),
                Shader.TileMode.REPEAT,
                Shader.TileMode.REPEAT
            )
        )
    }

    Box(
        modifier = modifier.background(brush)
    ) {
        /* ... */
    }
}
```

To save custom stateHolder we can create custom remember method for it
```kotlin
@Composable
private fun rememberMyAppState(
    windowSizeClass: WindowSizeClass
): MyAppState {
    return remember(windowSizeClass) {
        MyAppState(windowSizeClass)
    }
}

@Stable
class MyAppState(
    private val windowSizeClass: WindowSizeClass
) { /* ... */ }

```
