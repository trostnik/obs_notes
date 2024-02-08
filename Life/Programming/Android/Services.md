Service is an application component that is used to perform long-running operations, even if user switches to another application. It can run for some time after user closes app. Service can be accessed through any application component, even from other application. We can define Service private to change this.
### Foreground services
Foreground services are used to run tasks that are noticeable by user(for example playing music, downloading file). These services need to display notification that couldn't be dismissed, unless service stopped running.
### Background services
Background services are used for operations that are not necessary need to be known about(for example storage sorting). 
### Bound service
Bound service is a service to which any application components could be bound. It provides client-server interface, through which request can be sent and responses received. When all components unbound, service gets destroyed. We can define service both as bound and foreground(started, indefenitely-running). To create bound service we need to implement IBinder interface, which is returned by onBind() method and is used for communication between service and component. Bound service is used only to serve some component and destroyed by the client(calls unbindService()), when it's done working with it.  

There are two different possible lifecycles of service: bound and started.
![](https://developer.android.com/images/service_lifecycle.png)

onStartCommand() - the system invokes this method by calling startService(). When this method is called service is requested to run indefenitely. If we use this, we are responsible for stopping service either through stopSelf() or stopService(). 

onBind() - this method is called when the system calls bindService(). If this is called and onStartCommand() is not called service will run until it is bounded to at least one component. Return IBinder, needs to be implemented always, but if binding is not used null needs to be returned.

onCreate() - the system call this method one time for initial service setup before onStartCommand() and onBind()

onDestroy() - this method is called when service is no longer used and is being destroyed

Each service is declared in Manifest. We can provide description attribute that will display message in a list of services.

onStartCommand() returns Int. This Int defines how system should behave when service is killed

START_NOT_STICKY
If the system kills the service after onStartCommand() returns, do not recreate the service unless there are pending intents to deliver. This is the safest option to avoid running your service when not necessary and when your application can simply restart any unfinished jobs.
START_STICKY
If the system kills the service after onStartCommand() returns, recreate the service and call onStartCommand(), but do not redeliver the last intent. Instead, the system calls onStartCommand() with a null intent unless there are pending intents to start the service. In that case, those intents are delivered. This is suitable for media players (or similar services) that are not executing commands but are running indefinitely and waiting for a job.
START_REDELIVER_INTENT
If the system kills the service after onStartCommand() returns, recreate the service and call onStartCommand() with the last intent that was delivered to the service. Any pending intents are delivered in turn. This is suitable for services that are actively performing a job that should be immediately resumed, such as downloading a file.

To start a Service we can use startForegroundService(), which means that we create a promise that Service itself will call startForeground() within 5 seconds, but it is initially started in the background. Or we can use startService() which immediately starts background service. We can make multiple calls to startService(), it will trigger onStartCommand() each time, but Service will be the same and we need to call stopSelf() or stopService() only once.


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