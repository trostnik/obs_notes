Activity is the enter point to the application, an interactive screen on which user fully focuses.
The concept of activity is to provide a way of entering an app differently depending on the need of
user. For example if user enters from icon open main screen, but if users launches app from another
app that wants to use our app as a way to send message open message screen at the start. We need to
write the name of activity in manifest. Also there we can provide intent filters which allow us to
enter an activity from more general call(like SEND_MESSAGE(Implicit intent)). Activity that requests
other activity needs to have the same permissions.
Activity has lifecycle. Each state of licecycle is provided with a callback: onCreate(), onStart(),
onResume(), onPause(), onStop(), onDestroy(). 
onCreate() is used to make initialization of components that will be initialized only once. Binding
data to lists, initializing view, listeners etc.
onStart() is where user already sees the screen. It is the place where logic of maintaing UI needs to
be done. Starting animations, drawing some objects etc.
onResume() is where UI become interactive to the user. It is the place where we launch some 
functionality that need to run continuously(like camera preview).
onPause() is the first signal that activity starting to lose focus and prepared to be destroyed. 
Happens when user switches to another app in multi-tab mode, if dialog opens(when the activity is
partially or fully visible but not on the foreground). It can go further to onStop() or it can
continue running by returning to onResume(). It is the place where we need to pause long-running
operations(like camera preview) or release some resources that are not used noew
onStop() is the signal that activity is no longer visible to the user. It is the place where we need
to release resources that we were using and that are not already released. The state of UI components
are saved. App is continued from onStart(). If it goes further it gets to onDestroy().
onDestroy() is when Activity is preparing to be destroyed. It is the place where we need clear all
the resources. If user presses back or exits app we don't need to save anything but if we encounter
configuration change or destroying because of memory limit we need to implement logic to restore
user's previous state.
We can use startActivityForResult() to get result from another activity(for example take a photo and load it in another activity).
When configuratin change happens activity enters onPause(), onStop(), onDestroy(), onCreate(), onStart(), onResume(). To save data ViewModel(which survives configuration change), onSaveInstanceState() and persistent storage used. 
Configruration change happens on screen rotation, on multi-window switch, changing size of window, changing language, changing input device(keyboard).
When the dialog or activity, which partially covers the screen opens up, then activity enters onPause() state. If new activity takes the whole screen and pevious activity is no longer visible to the user then previous activity enters onStop() state. If user presses the home button activity behaves like it was completely covered.
