**BroadcastReceiver** is an Android component that works like publish-subscribe methos. It can subscribe to some sort of event of interest that may occur and respond to it.
Broadcasts can be **system-triggered** and **custom** for the application. Custom broadcasts can notify other application or components that may be interested in the event(for example, new data is downloaded).
Broadcast delivery times are not guaranteed, so applications shouldn't rely on it for communication.
Broadcast are packaged inside Intent and can contain additional information inside bundle.

There are some restrictions corresponding to Android version:
#### Android 14
While apps are in a [cached state](https://developer.android.com/guide/components/activities/process-lifecycle), broadcast delivery is optimized for system health. For example, less important system broadcasts such as [`ACTION_SCREEN_ON`](https://developer.android.com/reference/android/content/Intent#ACTION_SCREEN_ON) are deferred while the app is in a cached state. Once the app goes from the cached state into an [active process lifecycle](https://developer.android.com/guide/components/activities/process-lifecycle), the system delivers any deferred broadcasts.
Important broadcasts that are [declared in the manifest](https://developer.android.com/develop/background-work/background-tasks/broadcasts#manifest-declared-receivers) temporarily remove apps from the cached state for delivery.
#### Android 9
Beginning with Android 9 (API level 28), The [`NETWORK_STATE_CHANGED_ACTION`](https://developer.android.com/reference/android/net/wifi/WifiManager#NETWORK_STATE_CHANGED_ACTION) broadcast doesn't receive information about the user's location or personally identifiable data.
In addition, if your app is installed on a device running Android 9 or higher, system broadcasts from Wi-Fi don't contain SSIDs, BSSIDs, connection information, or scan results. To get this information, call [`getConnectionInfo()`](https://developer.android.com/reference/android/net/wifi/WifiManager#getConnectionInfo()) instead.
#### Android 8.0
Beginning with Android 8.0 (API level 26), the system imposes additional restrictions on manifest-declared receivers.
If your app targets Android 8.0 or higher, you cannot use the manifest to declare a receiver for most implicit broadcasts (broadcasts that don't target your app specifically). You can still use a [context-registered receiver](https://developer.android.com/develop/background-work/background-tasks/broadcasts#context-registered-receivers) when the user is actively using your app.
#### Android 7.0
Android 7.0 (API level 24) and higher don't send the following system broadcasts:
- `[ACTION_NEW_PICTURE](https://developer.android.com/reference/android/hardware/Camera#ACTION_NEW_PICTURE)`
- `[ACTION_NEW_VIDEO](https://developer.android.com/reference/android/hardware/Camera#ACTION_NEW_VIDEO)`
Also, apps targeting Android 7.0 and higher must register the `[CONNECTIVITY_ACTION](https://developer.android.com/reference/android/net/ConnectivityManager#CONNECTIVITY_ACTION)` broadcast using `[registerReceiver(BroadcastReceiver, IntentFilter)](https://developer.android.com/reference/android/content/Context#registerReceiver(android.content.BroadcastReceiver,%20android.content.IntentFilter))`. Declaring a receiver in the manifest doesn't work.

Apps can receive broadcast through **manifest-registered broadcasts** and **context-registered broadcasts**.

If your BroadcastReceiver is registered inside manifest than the system start your application when the broadcast is sent. 
```
Note: If your app targets API level 26 or higher, you cannot use the manifest to declare a receiver for _implicit_ broadcasts (broadcasts that do not target your app specifically), except for a few implicit broadcasts that are [exempted from that restriction](https://developer.android.com/guide/components/broadcast-exceptions). In most cases, you can use [scheduled jobs](https://developer.android.com/topic/performance/scheduling) instead.
```


