# Network Connectivity Example

A simple Android application demonstrating how to detect network connectivity status using Jetpack Compose and Kotlin.

## Features
- Displays network connectivity status ("Connected" or "Unavailable") in real-time.
- Utilizes Jetpack Compose for the UI.
- Implements a ConnectivityManager to observe network state changes.

## Screenshot
<div style="display: flex; justify-content: center; align-items: center;">
    <img src="https://github.com/user-attachments/assets/88e0de7c-5fe3-4465-a10c-bbdd8ebf6a32" alt="First Screenshot" style="width: 200px; height: auto; margin-right: 10px;">
    <img src="https://github.com/user-attachments/assets/52aa43f1-c9ef-4210-b636-c2106ea4226d" alt="Second Screenshot" style="width: 200px; height: auto;">
</div>

## Code Overview
The application consists of a `MainActivity` that sets the content view with a `NetworkScreen` composable. The NetworkScreen displays the network status text based on the current connectivity state.

**MainActivity**
The `MainActivity` sets up the content view and applies the theme.

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            NetworkConnectivityTheme {
                NetworkScreen()
            }
        }
    }
}
```

**NetworkScreen**
The `NetworkScreen` composable displays "Connected" or "Unavailable" based on the network connectivity state.

```kotlin
@Composable
fun NetworkScreen() {
    val connectionState by rememberConnectivityState()

    val isConnected by remember(connectionState) {
        derivedStateOf {
            connectionState === NetworkConnectionState.Available
        }
    }

    Box(
        modifier = Modifier.fillMaxSize(),
        contentAlignment = Alignment.Center
    ) {
        Text(
            text = if (isConnected) "Connected" else "Unavailable",
            fontSize = 48.sp,
            fontWeight = FontWeight.Bold
        )
    }
}
```

**Network Connectivity State**
The application uses a sealed interface to represent the network connection states:

```kotlin
sealed interface NetworkConnectionState {
    data object Available : NetworkConnectionState
    data object Unavailable : NetworkConnectionState
}
```

**Connectivity Manager**
The `ConnectivityManager` is used to observe the network connectivity state as a Flow:

```kotlin
private fun networkCallback(callback: (NetworkConnectionState) -> Unit): ConnectivityManager.NetworkCallback =
    object : ConnectivityManager.NetworkCallback() {
        override fun onAvailable(network: Network) {
            callback(NetworkConnectionState.Available)
        }

        override fun onLost(network: Network) {
            callback(NetworkConnectionState.Unavailable)
        }
    }

fun getCurrentConnectivityState(connectivityManager: ConnectivityManager): NetworkConnectionState {
    val network = connectivityManager.activeNetwork

    val isConnected = connectivityManager.getNetworkCapabilities(network)
        ?.hasCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET) ?: false

    return if (isConnected) NetworkConnectionState.Available else NetworkConnectionState.Unavailable
}

fun Context.observeConnectivityAsFlow(): Flow<NetworkConnectionState> = callbackFlow {
    val connectivityManager = getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager

    val callback = networkCallback { connectionState -> trySend(connectionState) }

    val networkRequest = NetworkRequest.Builder()
        .addCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET)
        .build()

    connectivityManager.registerNetworkCallback(networkRequest, callback)

    val currentState = getCurrentConnectivityState(connectivityManager)
    trySend(currentState)

    awaitClose {
        connectivityManager.unregisterNetworkCallback(callback)
    }
}

val Context.currentConnectivityState: NetworkConnectionState
    get() {
        val connectivityManager =
            getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager
        return getCurrentConnectivityState(connectivityManager)
    }
```

**Composable for Remembering Connectivity State**
The `rememberConnectivityState` composable provides a state holder for the network connection state:

```kotlin
@Composable
fun rememberConnectivityState(): State<NetworkConnectionState> {
    val context = LocalContext.current

    return produceState(initialValue = context.currentConnectivityState) {
        context.observeConnectivityAsFlow().collect {
            value = it
        }
    }
}
```

## Contributing

Contributions are welcome! Please fork the repository and submit a pull request for any improvements or bug fixes.

1. Fork the repository.
2. Create your feature branch (`git checkout -b feature/your-feature`).
3. Commit your changes (`git commit -am 'Add some feature'`).
4. Push to the branch (`git push origin feature/your-feature`).
5. Create a new Pull Request.

## Contact

For questions or feedback, please contact [@Bhavyansh03-tech](https://github.com/Bhavyansh03-tech).

---
