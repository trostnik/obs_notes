Navigation component - is an android framework which allows to navigate between app screens. 
Setup:
1. Create navigation graph(xml) inside navigation folder
2. Define destinations(screens inside app) 
3. Create actions(routes between destinations)
4. Create host inside main activity or any place(FragmentContainerView and set propery defaultNavHost = true and navGraph="nav_graph"). Add "name" attribute of NavHostFragment. FragmentContainerView will create instance of it and inflate it.
5. To navigate through screen use findNavController().navigateTo("destination_name")
6. To safely navigate use SafeArgs which encapsulate destinations and actions

We can create destinations from activities, fragments, dialogs. Activities could be launched both through implicit and explicit intents. \<argument\> is used to set dynamic arguments.
If we have destination that is entered from multiple locations we need to define global action.
If we need to implement some flow logic, we can use nested subgraphs and include them in main graph
NavController is placed in NavHostFragment(which we pass into FragmentContainerView).
When navigating through screens we can popUp some other screens from the backStack using popUpTo and popUpToInclusive attributes that take in screenId parameter. We can save state of poppedUp destinations using popUpToSaveState and restoreSaveState attributes
## Deep Links
##### Explicit deep link 
is a deep link that uses PendintIntent that is passed to the consumer and allows it to navigate to exact app location. To build such intent NavDeepLinkBuilder is used. When user navigate to destination using explicit intent the backstack of application gets cleared and startDestination and target destination is opened. When user pressed back he will be navigated to the default launch destination. By default NavDeepLinkBuilder uses launch Activity, but we can oveerride this, by using setComponentName method.
##### Imlicit deep link
is a deep link which allows user to navigate to your applications specific destination through URI, action and MIME types. To be able to use it we need to define \<deepLink\> attribute inside our destination in Navigation Graph and \<nav-graph\> attribute at the activity inside Manifest.  We can set flag Intent.FLAG_ACTIVITY_NEW_TASK  and navigation will behave like explicit deep linking(new task with cleared stack will be launced with target destination on top and default start destination), otherwise destination will be opened inside of the task of launching app. If we use standard launch mode on the activity deep links will be handled automatically, otherwise we need to call navController.handleDeepLink() inside onNewIntent() callback.
##### Animations
We can use  
app:enterAnim="@anim/slide_in_right"  
app:exitAnim="@anim/slide_out_left"  app:popEnterAnim="@anim/slide_in_left"  app:popExitAnim="@anim/slide_out_right" />
attributes to define different animations
