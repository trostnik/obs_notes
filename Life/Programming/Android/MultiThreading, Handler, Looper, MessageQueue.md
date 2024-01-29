We can run threads either by providing Runnable object to Thread class, or by inheriting from Thread class and overriding run() method.  When we call start() method on thread, the new thread is launched and now 2 threads are running concurrently (parent and child).

```kotlin
fun main() {
    val thread1 = Thread(CustomThread(), "thread1")
    CustomThread("thread2") 

    //Start the threads
    thread1.start()
    try { //delay for one second
        Thread.sleep(1000)
    } catch (e: InterruptedException) {
    }
    //Display info about the main thread
    println("Main Thread Calling.. "+ Thread.currentThread())
}

class CustomThread() : Runnable {
    lateinit var runner: Thread
    var name: String = "";

    constructor(threadName: String) : this() {
        name = threadName
        runner = Thread(this, name) // (1) Create a new thread.
        println("CustomThread Calling..  "+runner.name)
        runner.start()
    }

    override fun run() {
        //Display info about this particular thread
    println("CustomThread run calling.. " + Thread.currentThread());
    }
}
```

```kotlin
fun main() {
    val thread1 = Thread(CustomThread(), "thread1")
    CustomThread("thread2")

    //Start the threads
    thread1.start()

    try { //delay for one second
        Thread.sleep(1000)
    } catch (e: InterruptedException) {
    }
    //Display info about the main thread
    println("Main Thread Calling.. " + Thread.currentThread())
}

class CustomThread() : Thread() {

    constructor(threadName: String) : this() {
        name = threadName //set the thread name
        start()
    }

    override fun run() {
        //Display info about this particular thread
        println("CustomThread run calling.. " + currentThread());
    }
}
```

Thread Lifecycle: 
![[Pasted image 20230923093135.png]]
New: The thread is in the new state if you create an instance of Thread class but before the invocation of start() method
Runnable: The thread is in the runnable state after the invocation of the start() method, but the thread scheduler has not selected it to be the running thread.
Running: The thread is in running state if the thread scheduler has selected it.
Non-runnable: This is the state when the thread is still alive but is currently not eligible to run.
Terminated: 
**A thread is in a terminated or dead state when its run() method exits**.

<em>The application will not stop as long as at least one thread is still running even if the main thread stops running.</em>
### Daemon Thread
Daemon thread is a background thread that does not prevent the application from exiting if the main thread terminates.

Scenario-1: Background task that should not block our application for terminating (e.g File saving thread in the text editor)

Scenario-2: Code in the worker thread is not under our control and we do not want it to block our application for terminating. (e.g Worker thread that uses an external library)

thread.daemon=true

### What if One thread depends on another thread

→ Thread B[checking and goes to sleep] → Thread A [doing work, doing work, doing work, doing work,] →Wake up Thread B[checking(get the result)]
- **join():** It will put the current thread on wait until the thread on which it is called is dead. If a thread is interrupted then it will throw InterruptedException.
- **join(long millis)**: It will put the current thread on wait until the thread on which it is called is dead or wait for a specified time (milliseconds).
- **join(long millis, int nanos):** It will put the current thread on wait until the thread on which it is called is dead or wait for a specified time (milliseconds + nanos).
```kotlin
fun main() {
    val t1 = Thread(CustomRunnable(), "t1")
    val t2 = Thread(CustomRunnable(), "t2")
    val t3 = Thread(CustomRunnable(), "t3")
    t1.start()
    //start second thread after waiting for 2 seconds or if it's dead
    try {
        t1.join(2000)
    } catch (e: InterruptedException) {
        e.printStackTrace()
    }
    t2.start()
    //start third thread only when first thread is dead
    try {
        t1.join()
    } catch (e: InterruptedException) {
        e.printStackTrace()
    }
    t3.start()
    //let all threads finish execution before finishing main thread
    try {
        t1.join()
        t2.join()
        t3.join()
    } catch (e: InterruptedException) { // TODO Auto-generated catch block
        e.printStackTrace()
    }
    println("All threads are dead, exiting main thread")
}

class CustomRunnable : Runnable {
    override fun run() {
        println("Thread started:::" + Thread.currentThread().name)
        try {
            Thread.sleep(4000)
        } catch (e: InterruptedException) {
            e.printStackTrace()
        }
        println("Thread ended:::" + Thread.currentThread().name)
    }
}
```
## Multi-Threading
Multi-threadind is  a process of executing multiple threads simultaneously.

We use multithreading, rather than multiprocessing because threads use a shared memory area. They don’t allocate separate memory areas, though saving memory, and taking less time performing context-switching between the threads compared to the process.

As there is only one thread that updates the UI, which is the main thread, we use different other threads to do multiple tasks in the background but finally, to update the UI, we need to post the result to the main or UI thread.

It will be complicated to manage communication with all these threads if you are managing a large group of threads. So, Android has provided handlers to make the inter-process communication easier.

**The components of a Handler are:**

- Handler
	When you create a new handler, it is bound to the thread/message queue of the thread that is creating it — from that point on, it will deliver messages, runnable to that message queue and executes them as they come out of the message queue.
- Message
	The Message acts as a container for arbitrary data. The producer thread sends Messages to the Handler, which enqueues to the Message Queue. The Message provides three pieces of extra information, required by the Handler and Message Queue to process the message:

	what — an identifier the Handler can use to distinguish messages and process them differently
	time — informs the Message Queue when to process a Message
	target — indicates which Handler should process the Message
- Message Queue
	
	The MessageQueue is a queue that has a list of tasks (messages, runnable) that will be executed in a certain thread

	Android maintains a MessageQueue on the main thread.

	Alongside looper and handler, MessageQueues are part of the building blocks of threading in Android and they are used virtually everywhere in the system. Messages are not added directly to a MessageQueue, but rather through handler objects associated with the looper.
- Looper
	The looper is responsible for keeping the thread alive. It is a kind of worker that serves a MessageQueue for the current thread. Looper loops through a message queue and sends messages to corresponding threads to process.

## HandlerThread
It’s not necessary to create your own thread that has a Looper attached to it. Android provides a convenience class for this — “HandlerThread” It extends the Thread class and manages the creation of a Looper.

```kotlin
private var handler: Handler? = null
private var handlerThread: HandlerThread? = null
fun onCreate(@Nullable savedInstanceState: Bundle?) {
    super.onCreate()
    handlerThread = HandlerThread("HandlerDemo")
    handlerThread.start()
    handler = CustomHandler(handlerThread.getLooper())
}

fun onDestroy() {
    super.onDestroy()
    handlerThread?.quit()
}
```
### Thread Safety
We can create thread-local fields by assigning ThreadLocal instances to a field.
Let’s consider the following StateHolder class:

```java
public class StateHolder {
    
    private final String state;

    // standard constructors / getter
}
Copy
We can easily make it a thread-local variable:

public class ThreadState {
    
    public static final ThreadLocal<StateHolder> statePerThread = new ThreadLocal<StateHolder>() {
        
        @Override
        protected StateHolder initialValue() {
            return new StateHolder("active");  
        }
    };

    public static StateHolder getState() {
        return statePerThread.get();
    }
}
```

Thread-local fields are pretty much like normal class fields, except that each thread that accesses them via a setter/getter gets an independently initialized copy of the field so that each thread has its own state.

## Race condition
Read and Modify Operations: Multiple threads, let's call them Thread A and Thread B, want to read and modify the shared data. They both perform the following operations:

Read Operation: Both Thread A and Thread B read the current value of the shared data. This value is stored in their respective local caches or registers.

Modify Operation: Both Thread A and Thread B perform some calculations or updates based on the value they read from the shared data.

Thread A reads the initial value of the shared data into its local cache.
Thread B reads the same initial value into its local cache.
Thread A performs its calculations and updates the shared data with its result.
Thread B, unaware of the change made by Thread A, also performs its calculations based on the initial value it read and updates the shared data with its result.
## Synchronized
A piece of logic marked with synchronized becomes a synchronized block, allowing only one thread to execute at any given time.

We can use the synchronized keyword on different levels:

Instance methods
Static methods
Code blocks
When we use a synchronized block, Java internally uses a monitor, also known as a monitor lock or intrinsic lock, to provide synchronization. These monitors are bound to an object; therefore, all synchronized blocks of the same object can have only one thread executing them at the same time.
We can add the _synchronized_ keyword in the method declaration to make the method synchronized:

```java
public synchronized void synchronisedCalculate() {
    setSum(getSum() + 1);
}
```
Static methods are _synchronized_ just like instance methods:

```java
 public static synchronized void syncStaticCalculate() {
     staticSum = staticSum + 1;
 }
```
Sometimes we don’t want to synchronize the entire method, only some instructions within it. We can achieve this by _applying_ synchronized to a block:

```java
public void performSynchronisedTask() {
    synchronized (this) {
        setCount(getCount()+1);
    }
}
```
Notice that we passed a parameter _this_ to the _synchronized_ block. This is the monitor object. The code inside the block gets synchronized on the monitor object. Simply put, only one thread per monitor object can execute inside that code block.

If the method was _static_, we would pass the class name in place of the object reference, and the class would be a monitor for synchronization of the block:

```java
public static void performStaticSyncTask(){
    synchronized (SynchronisedBlocks.class) {
        setStaticCount(getStaticCount() + 1);
    }
}
```
## Volatile
The Java `volatile` keyword is used to mark a Java variable as "being stored in main memory". More precisely that means, that every read of a volatile variable will be read from the computer's main memory, and not from the CPU registers, and that every write to a volatile variable will be written to main memory, and not just to the CPU registers.
In a multithreaded application where the threads operate on non-volatile variables, each thread may copy variables from main memory into a CPU registers while working on them, for performance reasons. If your computer contains more than one CPU, each thread may run on a different CPU. That means, that each thread may copy the variables into the CPU registers of different CPUs. This is illustrated here:

![Threads may hold copies of variables from main memory in CPU caches.](https://jenkov.com/images/java-concurrency/java-volatile-1.png)

With non-volatile variables there are no guarantees about when the Java Virtual Machine (JVM) reads data from main memory into CPU registers, or writes data from CPU registers to main memory. This can cause several problems.
![[Pasted image 20230923200646.png]]
Actually, the visibility guarantee of Java `volatile` goes beyond the `volatile` variable itself. The visibility guarantee is as follows:

- If Thread A writes to a `volatile` variable and Thread B subsequently reads the same volatile variable, then all variables visible to Thread A before writing the volatile variable, will also be visible to Thread B after it has read the volatile variable.
- If Thread A reads a `volatile` variable, then all all variables visible to Thread A when reading the `volatile` variable will also be re-read from main memory.
### Atomic Classes

Пакет **java.util.concurrent.atomic** содержит девять классов для выполнения атомарных операций. Операция называется атомарной, если её можно безопасно выполнять при параллельных вычислениях в нескольких потоках, не используя при этом ни блокировок, ни синхронизацию [synchronized](https://java-online.ru/java-thread.xhtml#synchronized). Прежде, чем перейти к рассмотрению атомарных классов, рассмотрим выполнение наипростейших операций инкремента и декремента целочисленных значений.

С точки зрения программиста операции инкремента (i++, ++i) и декремента (i--, --i) выглядят наглядно и компактно. Но, с точки зрения JVM (виртуальной машины Java) данные операции не являются атомарными, поскольку требуют выполнения нескольких действительно атомарных операции: чтение текущего значения, выполнение инкремента/декремента и запись полученного результата. При работе в многопоточной среде операции инкремента и декремента могут стать источником ошибок. Т.е. в многопоточной среде простые с виду операции инкремента и декремента требуют использование синхронизации и блокировки. Но блокировки содержат массу недостатков, и для простейших операций инкремента/декремента являются тяжеловесными. Выполнение блокировки связано со средствами операционной системы и несёт в себе опасность приостановки с невозможностью дальнейшего возобновления потока, а также опасность взаимоблокировки или инверсии приоритетов (priority inversion). Кроме этого, появляются дополнительные расходы на переключение потоков. _Но можно ли обойтись без блокировок? В ряде случаев можно!_

Блокировка подразумевает **пессимистический** подход, разрешая только одному потоку выполнять определенный код, связанный с изменением значения некоторой «общей» переменной. Таким образом, никакой другой поток не имеет доступа к определенным переменным. Но можно использовать и **оптимистический** подход. В этом случае блокировки не происходит, и если поток обнаруживает, что значение переменной изменилось другим потоком, то он повторяет операцию снова, но уже с новым значением переменной. Так работают атомарные классы.

#### Описание атомарного класса AtomicLong

Рассмотрим принцип действия механизма **оптимистической блокировки** на примере атомарного класса AtomicLong, исходный код которого представлен ниже. В этом классе переменная value объявлена с модификатором _volatile_, т.е. её значение могут поменять разные потоки одновременно. Модификатор _volatile_ гарантирует выполнение отношения happens-before, что ведет к тому, что измененное значение этой переменной увидят все потоки.

Каждый атомарный класс включает метод **compareAndSet**, представляющий механизм _оптимистичной блокировки_ и позволяющий изменить значение value только в том случае, если оно равно ожидаемому значению (т.е. current). Если значение value было изменено в другом потоке, то оно не будет равно ожидаемому значению. Следовательно, метод compareAndSet вернет значение false, что приведет к новой итерации цикла while в методе getAndAdd. Таким образом, в очередном цикле в переменную current будет считано обновленное значение value, после чего будет выполнено сложение и новая попытка записи получившегося значения (т.е. next). Переменные current и next - локальные, и, следовательно, у каждого потока свои экземпляры этих переменных.

```java
private volatile long value;
 
public final long get() {
    return value;
}
 
public final long getAndAdd(long delta) {
    while (true) {
        long current = get();
        long next = current + delta;
        if (compareAndSet(current, next))
            return current;
    }
}
```

Метод **compareAndSet** реализует механизм оптимистической блокировки. Знакомые с набором команд процессоров специалисты знают, что ряд архитектур имеют инструкцию **Compare-And-Swap** (CAS), которая является реализацией этой самой операции. Таким образом, на уровне инструкций процессора имеется поддержка необходимой атомарной операции. На архитектурах, где инструкция не поддерживается, операции реализованы иными низкоуровневыми средствами.

Основная выгода от атомарных (CAS) операций появляется только при условии, когда переключать контекст процессора с потока на поток становится менее выгодно, чем немного покрутиться в цикле while, выполняя метод _boolean compareAndSwap(oldValue, newValue)_. Если время, потраченное в этом цикле, превышает 1 квант потока, то, с точки зрения производительности, может быть невыгодно использовать атомарные переменные.

## Synchronized and Concurrent Java Collections
Java platform provides strong support for handling multi-thread access through different synchronization _wrappers_ implemented within the _[Collections](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Collection.html)_ class.

These wrappers make it easy to create synchronized views of the supplied collections by means of several static factory methods.
```java
Collection<Integer> syncCollection = Collections.synchronizedCollection(new ArrayList<>());
    Runnable listOperations = () -> {
        syncCollection.addAll(Arrays.asList(1, 2, 3, 4, 5, 6));
    };
    
    Thread thread1 = new Thread(listOperations);
    Thread thread2 = new Thread(listOperations);
    thread1.start();
    thread2.start();
    thread1.join();
    thread2.join();
    
    assertThat(syncCollection.size()).isEqualTo(12);
}
```

As shown above, creating a synchronized view of the supplied collection with this method is very simple.

To demonstrate that the method actually returns a thread-safe collection, we first create a couple of threads.

After that, we then inject a [_Runnable_](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Runnable.html) instance into their constructors, in the form of a lambda expression. Let’s keep in mind that _Runnable_ is a functional interface, so we can replace it with a lambda expression.

Lastly, we just check that each thread effectively adds six elements to the synchronized collection, so its final size is twelve.
**Synchronized collections achieve thread-safety through [intrinsi](https://docs.oracle.com/javase/tutorial/essential/concurrency/locksync.html)[c locking](https://docs.oracle.com/javase/tutorial/essential/concurrency/locksync.html), and the entire collections are locked**. Intrinsic locking is implemented via synchronized blocks within the wrapped collection’s methods.

As we might expect, synchronized collections assure data consistency/integrity in multi-threaded environments. However, they might come with a penalty in performance, as only one single thread can access the collection at a time (a.k.a. synchronized access).
**Concurrent collections (e.g. _ConcurrentHashMap),_ achieve thread-safety by dividing their data into segments**. In a _ConcurrentHashMap_, for example, different threads can acquire locks on each segment, so multiple threads can access the _Map_ at the same time (a.k.a. concurrent access).

Concurrent collections are **much more performant than synchronized collections**, due to the inherent advantages of concurrent thread access.

## Executors
The Concurrency API introduces the concept of an `ExecutorService` as a higher level replacement for working with threads directly. Executors are capable of running asynchronous tasks and typically manage a pool of threads, so we don’t have to create new threads manually. All threads of the internal pool will be reused under the hood for revenant tasks, so we can run as many concurrent tasks as we want throughout the life-cycle of our application with a single executor service.

This is how the first thread-example looks like using executors:

```java
ExecutorService executor = Executors.newSingleThreadExecutor();
executor.submit(() -> {
    String threadName = Thread.currentThread().getName();
    System.out.println("Hello " + threadName);
});

// => Hello pool-1-thread-1
```

The class `Executors` provides convenient factory methods for creating different kinds of executor services. In this sample we use an executor with a thread pool of size one.

The result looks similar to the above sample but when running the code you’ll notice an important difference: the java process never stops! Executors have to be stopped explicitly - otherwise they keep listening for new tasks.

An `ExecutorService` provides two methods for that purpose: `shutdown()` waits for currently running tasks to finish while `shutdownNow()` interrupts all running tasks and shut the executor down immediately.

This is the preferred way how I typically shutdown executors:

```java
try {
    System.out.println("attempt to shutdown executor");
    executor.shutdown();
    executor.awaitTermination(5, TimeUnit.SECONDS);
}
catch (InterruptedException e) {
    System.err.println("tasks interrupted");
}
finally {
    if (!executor.isTerminated()) {
        System.err.println("cancel non-finished tasks");
    }
    executor.shutdownNow();
    System.out.println("shutdown finished");
}
```
In addition to `Runnable` executors support another kind of task named `Callable`. Callables are functional interfaces just like runnables but instead of being `void` they return a value.

This lambda expression defines a callable returning an integer after sleeping for one second:

```java
Callable<Integer> task = () -> {
    try {
        TimeUnit.SECONDS.sleep(1);
        return 123;
    }
    catch (InterruptedException e) {
        throw new IllegalStateException("task interrupted", e);
    }
};
```

Callables can be submitted to executor services just like runnables. But what about the callables result? Since `submit()` doesn’t wait until the task completes, the executor service cannot return the result of the callable directly. Instead the executor returns a special result of type `Future` which can be used to retrieve the actual result at a later point in time.

```java
ExecutorService executor = Executors.newFixedThreadPool(1);
Future<Integer> future = executor.submit(task);

System.out.println("future done? " + future.isDone());

Integer result = future.get();

System.out.println("future done? " + future.isDone());
System.out.print("result: " + result);
```
## Mutex and Semaphore
In a multithreaded application, two or more threads may need to access a shared resource at the same time, resulting in unexpected behavior. Examples of such shared resources are data-structures, input-output devices, files, and network connections.

We call this scenario a race condition. And, the part of the program which accesses the shared resource is known as the critical section. So, to avoid a race condition, we need to synchronize access to the critical section.

A mutex (or mutual exclusion) is the simplest type of synchronizer – it ensures that only one thread can execute the critical section of a computer program at a time.

To access a critical section, a thread acquires the mutex, then accesses the critical section, and finally releases the mutex. In the meantime, all other threads block till the mutex releases. As soon as a thread exits the critical section, another thread can enter the critical section.

**Synchronized** keyword,  is the simplest way to implement a mutex in Java

The _[ReentrantLock](https://www.baeldung.com/java-concurrent-locks)_ class was introduced in Java 1.5. It provides more flexibility and control than the _synchronized_ keyword approach.

Let’s see how we can use the _ReentrantLock_ to achieve mutual exclusion:

```java
public class SequenceGeneratorUsingReentrantLock extends SequenceGenerator {
    
    private ReentrantLock mutex = new ReentrantLock();

    @Override
    public int getNextSequence() {
        try {
            mutex.lock();
            return super.getNextSequence();
        } finally {
            mutex.unlock();
        }
    }
}
```
Like _ReentrantLock_, the _[Semaphore](https://www.baeldung.com/java-semaphore)_ class was also introduced in Java 1.5.

While in case of a mutex only one thread can access a critical section, _Semaphore_ allows **a fixed number of threads to access a critical section**. Therefore, **we can also implement a mutex by setting the number of allowed threads in a _Semaphore_ to one**.

Let’s now create another thread-safe version of _SequenceGenerator_ using _Semaphore_:

```java
public class SequenceGeneratorUsingSemaphore extends SequenceGenerator {
    
    private Semaphore mutex = new Semaphore(1);

    @Override
    public int getNextSequence() {
        try {
            mutex.acquire();
            return super.getNextSequence();
        } catch (InterruptedException e) {
            // exception handling code
        } finally {
            mutex.release();
        }
    }
}
```
So far, we’ve seen the options to implement mutex using features provided by Java.

However, the _Monitor_ class of Google’s Guava library is a better alternative to the _ReentrantLock_ class. As per its [documentation](https://guava.dev/releases/19.0/api/docs/com/google/common/util/concurrent/Monitor.html), code using _Monitor_ is more readable and less error-prone than the code using _ReentrantLock_.

First, we’ll add the Maven dependency for [Guava](https://mvnrepository.com/artifact/com.google.guava/guava):

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

Now, we’ll write another subclass of _SequenceGenerator_ using the _Monitor_ class:

```java
public class SequenceGeneratorUsingMonitor extends SequenceGenerator {
    
    private Monitor mutex = new Monitor();

    @Override
    public int getNextSequence() {
        mutex.enter();
        try {
            return super.getNextSequence();
        } finally {
            mutex.leave();
        }
    }

}
```
**Semaphore** is a synchronization primitive that is often used to control access to a shared resource among multiple threads. A semaphore maintains a set of permits, where each permit represents permission for one thread to access the resource. Semaphores are typically used to manage limited resources or to control the number of concurrent threads that can access a resource. Example of using Semaphore:
**Database Connection Pooling**: In many database-driven applications, you need to manage a limited number of available database connections. Creating a new database connection for every database operation can be inefficient and resource-intensive. Instead, you can use a connection pool.

In a database connection pool:

- A fixed number of database connections are created and made available in the pool.
- When a thread needs to perform a database operation, it requests a connection from the pool.
- The pool ensures that only a limited number of threads can access the database connections simultaneously.
- If all connections are in use, additional threads requesting a connection have to wait until a connection becomes available (limiting the number of concurrent database connections).
**Limiting Concurrent API Calls in a Weather App**: Imagine you're developing a weather application that retrieves weather data from a remote server. To ensure efficient use of network resources and prevent overloading the server, you can use a semaphore to limit the number of concurrent API calls made by your app.

Here's how this use case would work:

1. **API Request Limitation**: Weather data may be fetched from a third-party API, which has a rate limit on the number of requests your app can make in a given time frame. To comply with this rate limit and avoid penalties, you want to ensure that your app doesn't send too many concurrent requests.
    
2. **User Experience**: You also want to provide a smooth user experience. Limiting concurrent API calls ensures that the app doesn't make a large number of simultaneous requests, which could lead to slow responses and poor user experience.
    
3. **Resource Efficiency**: Controlling the number of concurrent API calls helps you manage network and server resources efficiently. It prevents overloading the server and reduces the risk of network congestion.
## DeadLock and LiveLock
Deadlock describes a situation where two or more threads are blocked forever, waiting for each other. Deadlock occurs when multiple threads need the same locks but obtain them in different order. A Java multithreaded program may suffer from the deadlock condition because the **synchronized** keyword causes the executing thread to block while waiting for the lock, or monitor, associated with the specified object. Here is an example.

Example
```java
public class TestThread {
   public static Object Lock1 = new Object();
   public static Object Lock2 = new Object();
   
   public static void main(String args[]) {
      ThreadDemo1 T1 = new ThreadDemo1();
      ThreadDemo2 T2 = new ThreadDemo2();
      T1.start();
      T2.start();
   }
   
   private static class ThreadDemo1 extends Thread {
   
      public void run() {
      
         synchronized (Lock1) {
            System.out.println("Thread 1: Holding lock 1...");

            try {
               Thread.sleep(10);
            } catch (InterruptedException e) {}
            System.out.println("Thread 1: Waiting for lock 2...");

            synchronized (Lock2) {
               System.out.println("Thread 1: Holding lock 1 & 2...");
            }
         }
      }
   }

   private static class ThreadDemo2 extends Thread {
   
      public void run() {
      
         synchronized (Lock2) {
            System.out.println("Thread 2: Holding lock 2...");
            
            try {
               Thread.sleep(10);
            } catch (InterruptedException e) {}
            System.out.println("Thread 2: Waiting for lock 1...");
            
            synchronized (Lock1) {
               System.out.println("Thread 2: Holding lock 1 & 2...");
            }
         }
      }
   } 
}
```
**Starvation and Livelock**
Starvation and livelock are much less common a problem than deadlock, but are still problems that every designer of concurrent software is likely to encounter.

**Starvation**
Starvation describes a situation where a thread is unable to gain regular access to shared resources and is unable to make progress. This happens when shared resources are made unavailable for long periods by "greedy" threads. For example, suppose an object provides a synchronized method that often takes a long time to return. If one thread invokes this method frequently, other threads that also need frequent synchronized access to the same object will often be blocked.
![[Pasted image 20230924085209.png]]


**Livelock**
A thread often acts in response to the action of another thread. If the other thread's action is also a response to the action of another thread, then livelock may result. As with deadlock, livelocked threads are unable to make further progress. However, the threads are not blocked — they are simply too busy responding to each other to resume work. This is comparable to two people attempting to pass each other in a corridor: Alphonse moves to his left to let Gaston pass, while Gaston moves to his right to let Alphonse pass. Seeing that they are still blocking each other, Alphone moves to his right, while Gaston moves to his left. They're still blocking each other, so...
![[Pasted image 20230924085046.png]]
