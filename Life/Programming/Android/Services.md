Service is an application component that is used to perform long-running operations, even if user switches to another application. It can run for some time after user closes app. Service can be accessed through any application component, even from other application. We can define Service private to change this.
### Foreground services
Foreground services are used to run tasks that are noticeable by user(for example playing music, downloading file). These services need to display notification that couldn't be dismissed, unless service stopped running.
### Background services
Background services are used for operations that are not necessary need to be known about(for example storage sorting). 
### Bound service
Bound service is a service to which any application components could be bound. It provides client-server interface, through which request can be sent and responses received. When all components unbound, service gets destroyed. We can define service both as bound and foreground(started, indefenitely-running). To create bound service we need to implement IBinder interface, which is returned by onBind() method and is used for communication between service and component. Bound service is used only to serve some component and destroyed by the client(calls unbindService()), when it's done working with it.  

**Priority** to less likely be killed by system:
1. Service bound to focused activity
2. Foreground service
3. Started Services

There are two different possible **lifecycles of service**: bound and started.
![](https://developer.android.com/images/service_lifecycle.png)

**onStartCommand()** - the system invokes this method aften another component called startService(). When this method is called service is requested to run indefenitely. If we use this, we are responsible for stopping service either through stopSelf() or stopService(). 

**onBind()** - this method is called when the system calls bindService(). If this is called and onStartCommand() is not called service will run until it is bounded to at least one component. Return IBinder, needs to be implemented always, but if binding is not used null needs to be returned.

**onUnbind(intent: Intent):** Boolean { - 
        // All clients have unbound with unbindService()
        return allowRebind
    }

**onRebind(intent: Intent)** {
	// A client is binding to the service with bindService(),
	// after onUnbind() has already been called
}

**onCreate()** - the system call this method one time for initial service setup before onStartCommand() and onBind()

**onDestroy()** - this method is called when service is no longer used and is being destroyed

Each service is declared in **Manifest**. We can provide description attribute that will display message in a list of services.

**onStartCommand() returns Int**. This Int defines how system should behave when service is killed

**START_NOT_STICKY**
If the system kills the service after onStartCommand() returns, do not recreate the service unless there are pending intents to deliver. This is the safest option to avoid running your service when not necessary and when your application can simply restart any unfinished jobs.
**START_STICKY**
If the system kills the service after onStartCommand() returns, recreate the service and call onStartCommand(), but do not redeliver the last intent. Instead, the system calls onStartCommand() with a null intent unless there are pending intents to start the service. In that case, those intents are delivered. This is suitable for media players (or similar services) that are not executing commands but are running indefinitely and waiting for a job.
**START_REDELIVER_INTENT**
If the system kills the service after onStartCommand() returns, recreate the service and call onStartCommand() with the last intent that was delivered to the service. Any pending intents are delivered in turn. This is suitable for services that are actively performing a job that should be immediately resumed, such as downloading a file.

To start a Service we can use **startForegroundService(),** which means that we create a promise that Service itself will call **startForeground() within 5 seconds**, but it is initially started in the background. Or we can use **startService() which immediately starts background service**. We can make multiple calls to startService(), it will trigger onStartCommand() each time, but Service will be the same and we need to call stopSelf() or stopService() only once.

If Service doesn't provide binding, then the Intent is the only form of communication between client and service. However, if we want **result back**, client can create **PendingIntent for a broadcast** (using [getBroadcast()](https://developer.android.com/reference/android/app/PendingIntent#getBroadcast(android.content.Context,%20int,%20android.content.Intent,%20int))) and deliver it to the service in the Intent that starts the service. The service can then use the broadcast to deliver a result.

We can perform **multiple requests** to service and onStartCommand will be called corresponding amount of times. But only single stopService() or stopSelf() is required to stop the service.

You **shouldn't use** **stopSelf() inside onStartCommand** callback when your service **handles multiple requests** because it will terminate all requests. **Instead you can use stopSelf(int)** which defines if request that was send is the last one and there are no request are running at the moment. That is, when you call stopSelf(int), you pass the ID of the start request (the startId delivered to onStartCommand()) to which your stop request corresponds. Then, if the service receives a new start request before you are able to call stopSelf(int), the ID doesn't match and the service doesn't stop.
We can **send notifications** through snackbar notifications or status bar notifications to inform user of something happening and provide additional action onClick on notification to open activity or something else.


To **implement Bound service** you have to implement onBind() method and provide your implementation of IBinder interface. Bound service lives as long as some component is bound to it. Create bound services when you want activity or other component interact with service or when you want to provide some functionality of your app to another applications through IPC. Component can call bindService() to bind to the service. bindService takes in implementation of ServiceConnection, which is used to communicate with the service through callbacks.
There are several ways to **implement IBinder**:
1. If your Service is used only by your application you can implement it by inhereting Binder class and provide instance of it inside onBind() method. Then, the component can use the implementation through accessing public methods.
2. Use a Messenger. Create Handler that handles different messages and provide it to Messenger. This allows your service to be used by other applications and perform IPC.
3. Use AIDL. Android Interface Definition Language (AIDL) decomposes objects into primitives that the operating system can understand and marshalls them across processes to perform IPC. The previous technique, using a `[Messenger](https://developer.android.com/reference/android/os/Messenger)`, is actually based on AIDL as its underlying structure. Create an .aidl file that defines the programming interface. The Android SDK tools use this file to generate an abstract class that implements the interface and handles IPC, which you can then extend within your service.

To work with **inherited Binder** you have to do the following:
- Create custom inner class that extends Binder and create public methods that would return instance of the service
- Get binder inside ServiceConnection in client component and get instance of service
- Call public methods of service
```kotlin
class LocalService : Service() {
    // Binder given to clients.
    private val binder = LocalBinder()

    // Random number generator.
    private val mGenerator = Random()

    /** Method for clients.  */
    val randomNumber: Int
        get() = mGenerator.nextInt(100)

    /**
     * Class used for the client Binder. Because we know this service always
     * runs in the same process as its clients, we don't need to deal with IPC.
     */
    inner class LocalBinder : Binder() {
        // Return this instance of LocalService so clients can call public methods.
        fun getService(): LocalService = this@LocalService
    }

    override fun onBind(intent: Intent): IBinder {
        return binder
    }
}

class BindingActivity : Activity() {
    private lateinit var mService: LocalService
    private var mBound: Boolean = false

    /** Defines callbacks for service binding, passed to bindService().  */
    private val connection = object : ServiceConnection {

        override fun onServiceConnected(className: ComponentName, service: IBinder) {
            // We've bound to LocalService, cast the IBinder and get LocalService instance.
            val binder = service as LocalService.LocalBinder
            mService = binder.getService()
            mBound = true
        }

        override fun onServiceDisconnected(arg0: ComponentName) {
            mBound = false
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.main)
    }

    override fun onStart() {
        super.onStart()
        // Bind to LocalService.
        Intent(this, LocalService::class.java).also { intent ->
            bindService(intent, connection, Context.BIND_AUTO_CREATE)
        }
    }

    override fun onStop() {
        super.onStop()
        unbindService(connection)
        mBound = false
    }

    /** Called when a button is clicked (the button in the layout file attaches to
     * this method with the android:onClick attribute).  */
    fun onButtonClick(v: View) {
        if (mBound) {
            // Call a method from the LocalService.
            // However, if this call is something that might hang, then put this request
            // in a separate thread to avoid slowing down the activity performance.
            val num: Int = mService.randomNumber
            Toast.makeText(this, "number: $num", Toast.LENGTH_SHORT).show()
        }
    }
}
```


In order to perform Inter-process communication you have to do the following:
1. Implement custom handler that would handle messages from other application
2. Create Messenger that would take handler as an argument
3. Pass binder that we can get from the messenger (Messenger.getBinder()) into onBind method of the service
4. Inside client application create ServiceConnection and create Messenger with serivce: IBinder received in onServiceConnected callback
5. Send messages to the service using created Messenger
6. Inside client application create custom Handler to implement two-way communication and pass it in replyTo field of the message in order the service to be able to send messages back/

```kotlin
class IPCService : Service() {

    companion object {
        private const val MSG_SET_NUMBER = 3
        const val IPC_ACTION = "com.example.coroutines.servicetest.IPC_ACTION"
    }


    internal class IPCHandler(private val context: Context,  looper: Looper = Looper.getMainLooper()) : Handler(looper) {
        override fun handleMessage(msg: Message) {
            when (msg.what) {
                MSG_SET_NUMBER -> {
                    val number = msg.data.getInt("DOUBLE", 0)
                    val messenger = msg.replyTo
                    messenger?.send(
                        Message.obtain(
                            null,
                            MSG_SET_NUMBER
                        ).apply {
                            data = bundleOf("DOUBLE" to number * number)
                        }
                    )
                    Toast.makeText(context, "Service is working!", Toast.LENGTH_SHORT).show()
                }

                else -> super.handleMessage(msg)
            }

        }
    }

    private val messenger: Messenger = Messenger(IPCHandler(this))


    override fun onBind(intent: Intent?): IBinder? {
        return messenger.binder
    }
}

class MainActivity : ComponentActivity() {

    private lateinit var editText: EditText
    private lateinit var button: Button
    private lateinit var textView: TextView



    private companion object {
        private const val MSG_RECEIVE = 3
    }

    class IPCHandler(private val textView: WeakReference<TextView>): Handler(Looper.getMainLooper()) {
        override fun handleMessage(msg: Message) {
            when(msg.what) {
                MSG_RECEIVE -> {
                    val value = msg.data.getInt("DOUBLE")
                    textView.get()?.text = value.toString()
                }
                else -> super.handleMessage(msg)
            }

        }
    }


    private var mService: Messenger? = null

    private var bound = false

    private val serviceConnection = object : ServiceConnection {
        override fun onServiceConnected(name: ComponentName?, service: IBinder?) {
            mService = Messenger(service)
            bound = true

        }

        override fun onServiceDisconnected(name: ComponentName?) {
            bound = false
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.main_activity)
        editText = findViewById(R.id.numberId)
        textView = findViewById(R.id.resultId)
        val messenger = Messenger(IPCHandler(WeakReference(textView)))
        button = findViewById<Button>(R.id.calculateButton).apply {
            setOnClickListener {
                Intent("com.example.coroutines.servicetest.IPCService").also {
                    it.`package` = "com.example.coroutines.servicetest"
                    bindService(it, serviceConnection, BIND_AUTO_CREATE)
                }
                if(bound) {
                    editText.text?.let {
                        sendMsgToService(Integer.valueOf(it.toString()), messenger)
                    }
                }
            }
        }
    }

    private fun sendMsgToService(msg: Int, messenger: Messenger) {
        val message = Message.obtain(null, MSG_RECEIVE).apply {
            data = bundleOf("DOUBLE" to msg)
        }
        message.replyTo = messenger
        mService?.send(message)
    }
}
```

Lifecycle of Bound Service
![[Pasted image 20240417155707.png]]

  
Some of the Service **usecases** from my experience:

to implement

1. location listener,
2. sound module, generating various voices
3. in app content updates,
4. API, provide services to other apps
5. in app billing
6. Communication with webservices (if requests frequency is high)
7. Music player
8. Downloading files
9. Live wallpapers,
10. Notification listeners
11. Screen savers
12. Input methods
13. Accessibility services