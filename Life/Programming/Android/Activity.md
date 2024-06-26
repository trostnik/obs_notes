Activity is the enter point to the application, an interactive screen on which user fully focuses.

The concept of activity is to provide a way of entering an app differently depending on the need of
user. For example if user enters from icon open main screen, but if users launches app from another
app that wants to use our app as a way to send message open message screen at the start. 

We need to write the name of activity in manifest. Also there we can provide intent filters which allow us to
enter an activity from more general call(like SEND_MESSAGE(Implicit intent)). Activity that requests
other activity needs to have the same permissions.

You define intent-filters for activities that you want to be accessible from another application. If you want activity to be accessible only from your own app, you can use explicit Intents (by Activity name).

Activity has lifecycle. Each state of licecycle is provided with a callback: onCreate(), onStart(),
onResume(), onPause(), onStop(), onDestroy(). 

**onCreate()** is used to make initialization of components that will be initialized only once. Binding
data to lists, initializing view, listeners etc.

**onStart()** is where user already sees the screen. It is the place where logic of maintaing UI needs to be done. It is the kind of logic that is non-sense without user seeing the screen and also logic that should be performed when the user comes back to activity. Starting video-playback,  Starting animations, Refreshing data from the network, from the services etc.

**onResume()** is where UI become interactive to the user. It is the place where we launch some functionality that need to run continuously(like camera preview). We need to decide whether the logic is needed to be running while we can't interact with the screen. If it is not, we can move this logic from onStart() to onResume().

**onPause()** is the first signal that activity starting to lose focus and prepared to be destroyed. 
Happens when user switches to another app in multi-tab mode, if dialog opens(when the activity is
partially or fully visible but not on the foreground). It can go further to onStop() or it can
continue running by returning to onResume(). It is the place where we need to pause long-running
operations(like camera preview) or release some resources that are not used noew

**onStop(****)** is the signal that activity is no longer visible to the user. It is the place where we need
to release resources that we were using and that are not already released. The state of UI components
are saved. App is continued from onStart(). If it goes further it gets to onDestroy().

**onDestroy()** is when Activity is preparing to be destroyed. It is the place where we need clear all
the resources. If user presses back or exits app we don't need to save anything but if we encounter
configuration change or destroying because of memory limit we need to implement logic to restore user's previous state.

We can use **startActivityForResult()** to get result from another activity(for example take a photo and load it in another activity).

When configuratin change happens activity enters onPause(), onStop(), onDestroy(), onCreate(), onStart(), onResume(). To save data ViewModel(which survives configuration change), onSaveInstanceState() and persistent storage used. 

There are 2 methods **onSaveInstanceState** and **onRestoreInstanceState**. OnSaveInstanceState is launched when activity might be temporarily destroyed, we can save some data to the bundle here and then receive it inside onCreate method. OnRestoreInstanceState is called after onStart() and receives the same bundle as onCreate(). Those methods only called during configuration changes and when the activity is destroyed due lack of memory. If we just press Back button these method won't be called.

Configruration change happens on screen rotation, on multi-window switch, changing size of window, changing language, changing input device(keyboard).

When the dialog or activity, which partially covers the screen opens up, then activity enters onPause() state. If new activity takes the whole screen and pevious activity is no longer visible to the user then previous activity enters onStop() state. If user presses the home button activity behaves like it was completely covered.

**Task** - is a collection of activities that user interacts with.
### Launch modes:
**standard**: behaves normally - every activity is placed on top of backstack
**singleTop**: if activity A on top of the stack and intent to activity A is happening again it is not created again but recalling and call onNewIntent()
**singleTask**: Activity is launched in separate task and behaves like singleTop
**singleInstance**: Activity is launched in singleTask and every other activities are opened in different task
"**singleInstancePerTask**" is always at the root of the activity task. Also, the device can hold only one instance of the "singleInstance" activity at a time, while the "singleInstancePerTask activity can be instantiated multiple times in different tasks when FLAG_ACTIVITY_MULTIPLE_TASK or FLAG_ACTIVITY_NEW_DOCUMENT is set.
Launch modes can be also passed to startActivity() and they will override those that written inside Manifest.

## Result Api
Result api is used instead of startActivityForResult because former was not stable due to the fact that activity that called other activity for result could have been destroyed and won't get handled. Result api register listener at the start of the lifecycle, so it is guaranteed to work. To implement this we need to use registerActivityForResult and provide it with ActivityResultContract(which defines input type of intent, and type of received data) and ActivityResultCallback(that defines actions onReceiveResult). registerActivityForResult can be launched only when Activity is at state CREATED and further.

## Task affinity
taskAffinity is attribute that defines behaviour of activity, what activity prefers to belong to(which task). If task affinity of bunch of activities are the same, they would prefer to stay in the same task. By default all activities prefer to stay in the same task, but it can be overriden, for example if we have more than one logical app inside our apk.

## Clearing a back stack after time
By default the back stack of task is cleared up to root activity after a long period of time. But we can override this by providing alwaysRetainTaskState,
clearTaskOnLaunch and finishOnTaskLaunch attributes. First, when defined in root activity, keeps back stack for long period of time, second when defined in root activity, clear all back stack except root activity even if use leaves up just for a moment, third is like second but for a single activity, and to whole task.

## Processes and Application Lifecycle
Android applications are running inside Linux processes and application doesn't have control over lifetime of process. It is controlled by system, so system has a set of rules that it follows with controlling lifetime of processes. To decide which process should be killed when low on memory, Android has "importance hierarchy". 
1. A foreground process(Activity onResume(), BroadcastReceiver onReceive(), Service onCreate(), onStart(), onDestroy()) - these processes will be killed only as last resort for freeing memory
2. A visible procceses(Activity onPause(), Service.startForeground()) - still extremely important
3. Service process(running Service.startService())
4. Cached process(Activity onStop) - system is free to kill them at any time 