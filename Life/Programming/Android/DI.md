Implementing dependency injection provides you with the following advantages:

- Reusability of code
- Ease of refactoring
- Ease of testing
Classes often depend on other classes. For example class `Car` depends on class `Engine`. There are 3 ways to provide the dependency:
- Instantiate class `Engine` inside `Car` class
- Get class from somewhere else. For example singleton, android `context` etc.
- Provide class `Engine` as a parameter of constructor of `Car` or as a parameter of `Car` methods which depend on class `Engine`
Third way is called dependency injection.
What are the problems with first way of providing dependency? 
1. It creates strong coupling which means that `Car` now can use only the exact instance created inside and cannot use other implementation. For example `GasEngine` or `ElectricEngine`. It makes code less flexible.
2. Makes testing harder, because `Car` uses real instance of `Engine` and you cannot mock it.
Android supports 2 types of dependency inejction
1. Constructor injection. Dependency provided on class creation
2. Field injection. There are classes in Android, such as Activity or Fragment, constructing of which are handled by the system, so you can't control and provide constructor parameters. For such cases you can inject dependency inside field:
```kotlin
     class Car {
    lateinit var engine: Engine

    fun start() {
        engine.start()
    }
}
```

#### Automated DI
When you are constructing dependencies yourself to pass to the constructor or parameters it is called manual DI. There are some problems that occur while using DI manually:
- In big applications there are complex hierarchy of dependency. If you want to implement DI yourself you will have to write a lot of boilerplate code.
- When you're not able to construct dependencies before passing them in — for example when using lazy initializations or scoping objects to flows of your app — you need to write and maintain a custom container (or graph of dependencies) that manages the lifetimes of your dependencies in memory.
Manual DI looks something like this 
```kotlin
class LoginContainer(val userRepository: UserRepository) {

    val loginData = LoginUserData()

    val loginViewModelFactory = LoginViewModelFactory(userRepository)
}

// AppContainer contains LoginContainer now
class AppContainer {
    ...
    val userRepository = UserRepository(localDataSource, remoteDataSource)

    // LoginContainer will be null when the user is NOT in the login flow
    var loginContainer: LoginContainer? = null
}

...

class LoginActivity: Activity() {

    private lateinit var loginViewModel: LoginViewModel
    private lateinit var loginData: LoginUserData
    private lateinit var appContainer: AppContainer


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        appContainer = (application as MyApplication).appContainer

        // Login flow has started. Populate loginContainer in AppContainer
        appContainer.loginContainer = LoginContainer(appContainer.userRepository)

        loginViewModel = appContainer.loginContainer.loginViewModelFactory.create()
        loginData = appContainer.loginContainer.loginData
    }

    override fun onDestroy() {
        // Login flow is finishing
        // Removing the instance of loginContainer in the AppContainer
        appContainer.loginContainer = null
        super.onDestroy()
    }
}

```
There are libraries that perform these actions for you. They fit into 2 categories
1. Reflection-based runtime libraries
2. Static compile-time libraries
#### Service Locator
As an alternative to dependency injection there is a Service Locator pattern. You create a class that stores all dependency and provides them on demand.
The difference between DI and Service Locator is how classes get their dependencies. In the DI application controlls providing dependencies, in Service Locator classes themselfs control and ask for dependency from Service Locator.
```kotlin
object ServiceLocator {
    fun getEngine(): Engine = Engine()
}

class Car {
    private val engine = ServiceLocator.getEngine()

    fun start() {
        engine.start()
    }
}

```

It creates several problems:
- Testing becomes harder because classes have to work with global ServiceLocator class and mock all the methods of this class and all tests share the same instance of ServiceLocator because it is a Singleton.
- Class is using the concrete implementation of dependency. Because of that it is harder to understand what exact implementation class need from the outside. As a result, changes to `Car` or the dependencies available in the service locator might result in runtime or test failures by causing references to fail.
- Managing lifetimes of objects is more difficult if you want to scope to anything other than the lifetime of the entire app.

#### Dagger

Dagger generated dependency injection code, similar to manual DI in compile time. To let Dagger create factories annotate class and all it's dependencies with @Inject:
```kotlin
// @Inject lets Dagger know how to create instances of this object
class UserRepository @Inject constructor(
    private val localDataSource: UserLocalDataSource,
    private val remoteDataSource: UserRemoteDataSource
) { ... }

// @Inject lets Dagger know how to create instances of these objects
class UserLocalDataSource @Inject constructor() { ... }
class UserRemoteDataSource @Inject constructor() { ... }
```

@Component annotation generates dependency graph which then can be referenced through creating implementation of component:
```kotlin
// @Component makes Dagger create a graph of dependencies
@Component
interface ApplicationGraph {
    // The return type  of functions inside the component interface is
    // what can be provided from the container
    fun repository(): UserRepository
}

...

// Create an instance of the application graph
val applicationGraph: ApplicationGraph = DaggerApplicationGraph.create()
// Grab an instance of UserRepository from the application graph
val userRepository: UserRepository = applicationGraph.repository()

```
