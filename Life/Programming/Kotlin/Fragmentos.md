To add Fragment we need to use add(), providing it with id of container and class name of Fragment
To remove Fragment we need to use remove(), providing it with Fragment, found using findFragmentById() or findFragmentByTag()
To replace Fragment we need to use replace, providing it with id of container and class name of Fragment
It is recommended to use class name instead of instance to allow saving state work properly
commit() is operation that  commits all transactions. It is asynchronous, meaning that it will schedule all transaction to the time, main thread is ready. To perform all transactions immediately we can use commitNow(), but transactions then are not allowed to be added to backStack.
We can limit lifecycle of added fragment using setMaxLifecycle() to allow fragment only reach provided lifecycle state, and if it goes further it will be forced to the provided lifecycle state
show() and hide() methods are FragmentTransaction method which make view of fragment visible or invisible. They are use when you want visibility changes to be associated with transactions and added to backStack
detach() method detaches the view, which means it gets destroyed and Fragment goes to STOP state. attach() method recreates the view, attached and displayed. If we call attach and detach methods in the same transaction on the same Fragment it will get efficiently handled and not destroyed. To attach and reatach fragment immediately we need to use different transactions separeted by executePendingTransactions()

## Fragment Lifecycle
Each Fragment has it's own lifecycle. It implements LifecycleOwner. Also each Fragment has control over lifecycle of attached view. FragmentManager handles the fragment lifecycle and process of attaching and detaching to host. Fragment has onAttach() and onDetach() methods that are executed before and after all lifecycle events respectively.

![fragment lifecycle states and their relation both the fragment's
lifecycle callbacks and the fragment's view lifecycle](https://developer.android.com/images/guide/fragments/fragment-view-lifecycle.png)
Fragment CREATED. The place where savedInstanceState is restored. The view is not initialized in this state.
Fragment CREATED and View INITIALIZED. This is the appropriate place to set up the initial state of your view, to start observing LiveData instances whose callbacks update the fragment's view, and to set up adapters on any RecyclerView or ViewPager2 instances in your fragment's view.
Fragment and View STARTED. The view of fragment is available.
Fragment and View RESUMED. The fragment is fully available.
Fragment CREATED and View DESTROYED. The fragment's view has reached the end of its lifecycle and getViewLifecycleOwnerLiveData() returns a null value. At this point, all references to the fragment's view should be removed, allowing the fragment's view to be garbage collected.
Fragment DESTROYED. The fragment reached the end of the lifecycle.
## Saving state with fragments
There are different data on display which state saving needs to be handled.
Variables, SavedState, ViewState, NonConfig.
Variables are saved through SavedState. SavedState is a serializable portion of data that can be put inside Bundle onSaveInstanceState and retrieved  in your fragment's onCreate(Bundle), (LayoutInflater, ViewGroup, Bundle), and onViewCreated(View, Bundle) methods.
View is responsible for saving it's own View state, we don't need to implement anything, we just need to define view id.
NonConfig data is a data that received from third party like from network or from db. This kind of data needs to be stored inside ViewModel which handles saving state.

## Comunicating between fragments
There are two ways of comunicating between fragments: 
1. Through shared ViewModel
2. Through Fragment Result Api
When comunicating through shared ViewModel we need to use viewModel that is inside mutual scope. So when we need to comunicate between host Activity and fragment we define viewModel in activity and retrieve it inside fragment by using activity scope. When we need to comunicate between Fragments we can use ViewModel in activity scope. 
When comunicating throuh Fragment Result Api wee need to define FragmentResultListener and setFragmentResult methods of FragmentManager. When we need to pass data between parent and child fragments, parent fragment needs to use childFragmentManager.