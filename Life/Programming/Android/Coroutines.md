Every coroutine needs to be launched inside some CoroutineScope. CoroutineScope contains Coroutine
Context, which consists of Dispatcher and Job. There are extension functions to the CoroutineScope
which handle the execution of coroutines(launch, async, await).
When suspend function or coroutine is launched it gets suspended and control goes to the other part
of function which is running until another suspend function is called, after that launched coroutine
gets control and executes the code
launch{} launches the coroutine. It is extension function of CoroutineScope so it can be used only
within it. Returns a Job, which can be cancelled, paused, resumed, waited for etc. Job can be part
of hirarchy of jobs(If child Job gets cancelled or throw exception, parent job gets cancelled, if
SupervisorJob is not used)
coroutineScope{} is a suspend function that takes block of code and runs it in inherited Coroutine
Scope, but overrides Job. It is used to create function that launched multiple suspend functions in parallel.
To cancel coroutine we need to check inside it that it still is active, because if we don't check and there is some computation happening inside
it just won't be cancelled. Cancellable suspend functions(standard kotlin library functions) are 
cancellable and if we call them inside cancelled coroutine they throw CancellationException which
can be handled. But inside finally block we cannot use any suspend functions(because coroutine is
already cancelled). To overcome this we can use NonCancellable context. 
By default coroutines are sequential which means that they will execute one after another and if we
use them as usual functions that return results they will execute one after another, not in parallel.
To execute suspend functions that return something in parallel we need to use async{} .await(). Async creates a coroutine, but unlike the usual one it returns Deffered instead of Job. This Deffered 
contains some result which then can be received using await(). Hence we can run multiple functions
simultaneously and receive their results all at once. The time is defined by the longest running
operation. We can use async(start = CoroutineStart.LAZY) to delay the execution of the function.
It will be executed only when we use .await() or .start(). It is used to delay some heavy operation
but that means that it takes more time if there are multiple functions because they will run 
sequentially. To run them in parallel we can use .start(). We can create async style functions which
return Deffered and they will not be suspend, but to get results we need CoroutineScope, because we
use .await() which is suspend function. If we use these type of functions we may encounter leak, when
one coroutine is launched and then error happens, coroutine doesn't get cancelled. To overcome this
we need to use suspend functions which contain their scope(coroutineScope). It is followed by principle
of structured concurrency and that means when child coroutine fails all other coroutines get cancelled
and parent coroutine also get cancelled.
Coroutine always execute in some Context. Context contains of different elements. Main elements are
Dispatcher and Job. Dispatcher is abstraction over threads which tells how coroutine will be executed
(on a new thread, on exact thread, on different threads). There are Default, Unconfinded, Main, IO.
Any UI related work need to be done on Main dispatcher. Default is for heavy computational work.
IO is for input output operations like reading, writing to db or from network. Unconfined is dispatcher
that starts on caller thread and then switched to thread on which first suspend function executed.
Used for non-heavy CPU usage work and for non strict bound to thread(like main). 
Coroutines could be debugged using Android Studio debugger. We can acces them through log which will
print current thread and coroutine name. We can switch threads using withContext(thread) in coroutines.
We can receive Job object of coroutine because it is part of the context using coroutineContext[Job].
By default coroutine inherits parent's Dispatcher and create Job which is integrated in hierarchy of
Jobs. That means if parent coroutine gets cancelled all child coroutines get cancelled as well. However
we can override this by providing parameter Job() to coroutine builder. The job will not be the part of
the hierarchy and will run independently. Parent coroutine waits for completion of all children. We can
name coroutine using CoroutineName and combine context elements using + operator. If we need to bound
coroutines to some element with lifecycle we can create our own scope using CoroutineScope() or 
MainScope(). Then we can cancell this scope onDestroy() and all coroutines will be cancelled as well.
Flow is a sequence of data which is emitted asynchronously. To create flow you don't need to use 
suspend keyword. Flows are cold which means they start emitting values only after someone starts to 
collect them. Flows checks itself to be active on every emit and throw CnancellationError when cancelled.
We can build flow using flow{} or .asFlow for any sequential set. There are all standart intermediate
operators in flow like in RxJava(map, flatMap, filter,flatten etc.). There is also transform operator
which is general operator for any transformation which can be used to create own transforming operators
like filter, map. There are limiting operators(take(3)), terminal operators(which is used to collect the
flow items(reduce, collect, toList, toSet, first, fold)). Flows are sequential which means they will
emit items sequentially one after another unless operators that work with multiple flows are used
(flatMap). By default flow is executed in caller thread, which means that any computation inside will
be done in caller thread and it will collected on caller thread as well. To change thread we need to 
use flowOn(Dispatcher) operator, which change computation but collection is happening on caller thread.
We can use buffer when we have slow flow handler to save emitted items and pass them as fast as handler
can handle them instead of waiting for handler to handle first. We can use conflate to store only recent
items that managed to get through handler, all other items will be conflated(deleted). collectLatest is
used like usual collect but it will drop items that were emitted when handler was in process of handling
Zip function will combine corresponding values of two flows, other items from flows will be lost.
Combine function will combine lates values from one flow with latest values from another flow.
flatMapConcat will transform Flow<Flow\<T\>> in Flow\<T\> and perform some mapping and then emit items
sequentially(wait for previous inner flow to complete and then start next one)
flatMapMerge will convert items into a single flow and emit items as soon as possible(it won't wait for
previous emits, it will perceive all emits as a part of one flow)
flatMapLatest is like collectLatest(collection of previous flow is cancelled when new flow emitted) 
To handle errors from flows we can use try{}catch{}, or we can use .catch{} operator.
## Channels
Channels are used to receive and send a stream of values. They are like subjects from RxJava. send and receive functions are suspend functions and needed to be launched in CoroutineScope. Channels can be closed to be sure they stopped sending items. There are channel builders. One of them - produce. It returns ReceiveChannel and it's scope implements Channel interface. Pipeline is a pattern where producer produce potentially infinite number of items and consumer waits for them. If multiple coroutines listen to the same channel they cooperate to get items. Each item consumed only once. If multiple coroutines send to the same channel channel consumes all the items with respect of the order of execution. By default channels work like rendezvous(they wait for each other. if sender sends, but receiver is busy it gets suspended and vice versa). But we can define buffer for Channel and it values will be sent to buffer and receiver will process them as fast as it can(without waiting for sending processing).  Ticker channels are rendezvous channels that send Unit each given period.
## Exception Handling
Coroutines builders can either propogate uncaught errors(pass the error to the next builder, throw error immediately): launch(), actor() or wait for the user to handle them(through await() or receive()).  CoroutineExceptionHandler is used to override behaviour when facing uncaught errors. It doesn't recover from exceptions, when it is launched exception already happened, it is used to log additional info, restart app, terminate etc. We need to define exceptionHandler if we want only for the parent scope, because all child coroutines will cancel parent coroutine with caught exception and it does not make sense to define CoroutineExceptionHandler for them.
Cancellation is coroutines is bidirectional(if child fails parent gets cancelled, if parent fails children get cancelled). But we can use SupervisorJob() to override this. If we provide CoroutineScope with SupervisorJob(), then if child coroutine gets cancelled, parent coroutine won't. There is also a builder for this supervisorScope{}. If we use SupervisorJob() we need to handle all child coroutines because exceptions won't propogate to parent.
## Shared mutable state
When multiple coroutines launched and they change the state of some shared variable it won't get expected result, because they run without any synchronization. To be able to synchronize them we need to use either single thread for making changed(withContext(singleThreadContext)) or we can use Mutex(). Mutex controlls the behaviour of changing coroutines and they get executed in controlled way. There is mutex.withLock builder that create block mutex.lock(); try { ... } finally { mutex.unlock() }. 
Actor is an entity which contains state, channel for communication and behaves as coroutine. It can provide shared mutable state with thread safety, but it can be changed only through messages of the channel. There is builder for actor: actor\<T\>{} which takes type of message, that will be send to and from actor. 
## StateFlow and SharedFlow
StateFlow is a hot flow which emit latest value and each subsequent value to consumer. It can be used to present mutual state flow, using MutableStateFlow. StateFlow lives in memory for as long as there is reference to it. StateFlow requires initial value and it doesn't unregister consumer when the VIEW goes to STOPPED state.To achieve the same behavior,you need to collect the flow from a `Lifecycle.repeatOnLifecycle` block( repeatOnLifecycle launches the block in a new coroutine every time the  lifecycle is in the STARTED state (or above) and cancels it when it's STOPPED). We can use .shareIn  operator to transform any cold flow to hot flow. It takes coroutineScope, replayNumber and start behaviour policy(when producer starts emitting and how it will behave)
SharedFlow is hot flow and it is a generalization of StateFlow. It doesn't take initial default value and doesn't emit nothing by default, unlike StateFlow.