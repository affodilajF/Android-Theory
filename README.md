# ROUND TWO : UI ARCHITECTURE
- Thinking in Compose
- Lifecycle 
- Side effect

https://medium.com/@mortitech/exploring-side-effects-in-compose-f2e8a8da946b
https://developer.android.com/develop/ui/compose/side-effects


# Thinking in Compose ----- 
## The best practice is to keep your composable functions fast, idempotent, and side-effect free.
### A. Recomposition 
- In imperative UI model, to change a widget, you call a setter on the widget to change its internal state. In Compose, you call composable function again with NEW DATA.
Doing so caouses the function to be recomposed-- the widgets emitted by the function are redrawn, if necessary, with new data.

- RECOMPOSING THE ENTIRE UI TREE CAN BE COMPUTATIONALLY EXPENSIVE (computing power and battery life). Compose sloves this with intellegent recomposition. How? When Compose recomposes based on new inputs, it only calls the functions or lambdas that might have changed, and skips the rest. By skipping all functions or lambdas that don't have changed parameters, Compose can recompose efficiently.

- NEVER DEPEND ON SIDE-EFFECT FROM EXECUTING COMPOSABLE FUNCTIONS, since function's recomposition may be skipped. If you do, user may be experiencing strange and unpredictable behaviour in your app.
- A side-effect is ANY CHANGE THAT IS VISIBLE TO THE REST OF YOUR APP. For example :
  - Writing to a property of a shared object.
  - Updating an observable in ViewModel.
  - Updating shared preferences.

Best Partices : 
Read and write do not occour inside composable. Instead, do it inside ViewModel (in coroutine background) and to handle UI logic, pass with current value with a callback to trigger an update (in this case, isChecked)

```kotlin
class MyViewModel : ViewModel() {
    private val _isChecked = mutableStateOf(false)
    val isChecked: State<Boolean> get() = _isChecked

    fun onCheckboxChanged(isChecked: Boolean) {
        _isChecked.value = isChecked
        // Perform background operation to update SharedPreferences
        viewModelScope.launch {
            // Code to write `isChecked` to SharedPreferences
        }
    }
}

@Composable
fun MyScreen(viewModel: MyViewModel) {
    SharedPrefsToggle(
        text = "Enable Feature",
        value = viewModel.isChecked.value,
        onValueChanged = { newValue -> viewModel.onCheckboxChanged(newValue) }
    )
}

@Composable
fun SharedPrefsToggle(
    text: String,
    value: Boolean,
    onValueChanged: (Boolean) -> Unit
) {
    Row {
        Text(text)
        Checkbox(checked = value, onCheckedChange = onValueChanged)
    }
}
```

Incorrect partices:

Issues =>
- Direct Side Effects in composable. Performing side effect like updating SharedPreferences directly within a composable can lead to unintended behaviour and make composable harder to test and maintain.
- Performance Problems. IDK ???? OH it can lead to UI Freezes => Heavy computations or I/O operations in composables can block the main thread, causing the UI to freeze or become unresponsive.
- Poor sepaeation of concerns. Mixing UI logic with data persistence logic violates the principle of separation of concerns.
- Potential for Repeated Writes. Well, unintended and strange behaviour we can predict.

```kotlin
@Composable
fun SharedPrefsToggle(
    text: String,
    value: Boolean,
    sharedPreferences: SharedPreferences
) {
    Row {
        Text(text)
        Checkbox(
            checked = value,
            onCheckedChange = { newValue ->
                // Directly writing to SharedPreferences inside the composable
                sharedPreferences.edit().putBoolean("key", newValue).apply()
            }
        )
    }
}

```

### B. Composable functions can execute in any order
The calls to StartScreen, MiddleScreen, and EndScreen might happen in any order. This means you can't, for example, have StartScreen() set some global variable (a side-effect) and have MiddleScreen() take advantage of that change. Instead, each of those functions needs to be self-contained.

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

### C. Composable functions can run in parallel
This lets Compose take advantage of multiple cores, and run composable functions not on the screen at a lower priority.

### D. Recomposition is optimistic
Means compose expect to finis recomposition before the params change again. If the params does change before recomposition finishes, Compose might CANCEL THE RECOMPOSITION and restart it with the new parameter. 

When recomposition is canceled, Compose discards the UI tree from the recomposition. If you have any side-effects that depend on the UI being displayed, the side-effect will be APPLIED EVEN IF THE COMPISITION IS CANCELED. This can lead to inconsistent app state.

# Lifecycle -----

# Side Effect -----
Ia s change to the state of the app that happens outside the scope of a composable function. 

Composable should be side-effecr free. When you need to make changes to the state of the app, you should use the Effect API so that those side effects are executed in a predictable manner. 

Key term: An effect is a composable function that doesn't emit UI and causes side effects to run when a composition completes. 


### Side Effect
- If recomoposed process happening inside exact composable, the side effect will trigger.
- The side effect triggers only when the current composable function is recomposed ant NOT FOR ANY NESTED COMPOSABLE FUNCTIONS.

### Launchedffect
- Have 2 parameter.
  - KEY param => If the key param is changed, then LaunchedEffect will trigger.
  - Block param => lambda that defines the side effect to be executed.
- Is a Composable function that executes a side effect in a separate coroutine scope (therfore we can run suspend function there).
- It launches a coroutine with the block of the code passed as parameter.
- The coroutine will be cancelled if LaunchedEffect leaves the composition.
- Proper to execute long operations (network calls, animations) without blocking the UI threads.

### DiposableEffect 
- DisposableEffect is a Composable function that executes a side effect when its parent Composable is first rendered, and disposes of the effect when the Composable is removed from the UI hierarchy.
- This function is useful for managing resources that need to be cleaned up when a Composable is no longer in use.

```kotlin
    DisposableEffect(Unit) {
        println("Setup: Composable is being composed")

        onDispose {
            println("Cleanup: Composable is being removed or key changed")
        }
    }
```


      
    
