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