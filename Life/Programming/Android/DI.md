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

@Component annotation generates dependency graph which then can be referenced through creating implementation of component. Dependency graph is just a container with factories of all needed dependency (annotated with @Inject or provided inside @Module):
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

In Android you have to define Application Graph inside custom Application class because you want the graph to be accessible across the whole app and during whole application lifecycle:
```kotlin
// Definition of the Application graph
@Component
interface ApplicationComponent { ... }

// appComponent lives in the Application class to share its lifecycle
class MyApplication: Application() {
    // Reference to the application graph that is used across the whole app
    val appComponent = DaggerApplicationComponent.create()
}
```

In order to inject Android component, which lifecycle is not controlled directly, we can use Field Injection. Provide Dagger with component that needs to be injected and declare all the dependency and call inject function of ApplicationComponent. This component will generate container with dependencies needed to satisfy all @Inject fields inside Activity:
```kotlin
@Component
interface ApplicationComponent {
    // This tells Dagger that LoginActivity requests injection so the graph needs to
    // satisfy all the dependencies of the fields that LoginActivity is requesting.
    fun inject(activity: LoginActivity)
}

class LoginActivity: Activity() {
    // You want Dagger to provide an instance of LoginViewModel from the graph
    @Inject lateinit var loginViewModel: LoginViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        // Make Dagger instantiate @Inject fields in LoginActivity
        (applicationContext as MyApplication).appComponent.inject(this)
        // Now loginViewModel is available

        super.onCreate(savedInstanceState)
    }
}

// @Inject tells Dagger how to create instances of LoginViewModel
class LoginViewModel @Inject constructor(
    private val userRepository: UserRepository
) { ... }

class UserRepository @Inject constructor(
    private val localDataSource: UserLocalDataSource,
    private val remoteDataSource: UserRemoteDataSource
) { ... }

class UserLocalDataSource @Inject constructor() { ... }
class UserRemoteDataSource @Inject constructor(
    private val loginService: LoginRetrofitService
) { ... }
```

In the example abover LoginRetrofitService is used. In order to construct this dependency, we have to use Retrofit library's Builder. We don't own classes from Retrofit library. To provide this kind of dependencies we need to use Modules. 

```kotlin
// @Module informs Dagger that this class is a Dagger Module
@Module
class NetworkModule {

    // @Provides tell Dagger how to create instances of the type that this function
    // returns (i.e. LoginRetrofitService).
    // Function parameters are the dependencies of this type.
    @Provides
    fun provideLoginRetrofitService(): LoginRetrofitService {
        // Whenever Dagger needs to provide an instance of type LoginRetrofitService,
        // this code (the one inside the @Provides method) is run.
        return Retrofit.Builder()
                .baseUrl("https://example.com")
                .build()
                .create(LoginService::class.java)
    }
}

// The "modules" attribute in the @Component annotation tells Dagger what Modules
// to include when building the graph
@Component(modules = [NetworkModule::class])
interface ApplicationComponent {
    ...
}
```

We can also provide these methods with needed dependencies:
```kotlin
@Module
class NetworkModule {
    // Hypothetical dependency on LoginRetrofitService
    @Provides
    fun provideLoginRetrofitService(
        okHttpClient: OkHttpClient
    ): LoginRetrofitService { ... }
}
```

Scopes are the way to bound lifecycle of class to lifecycle of the component. It is used when you want to use the same unique instance in different places of application or when the constructing of the element is expensive and same instance can be used.  
```kotlin
@Singleton
@Component(modules = [NetworkModule::class])
interface ApplicationComponent {
    fun inject(activity: LoginActivity)
}

@Singleton
class UserRepository @Inject constructor(
    private val localDataSource: UserLocalDataSource,
    private val remoteDataSource: UserRemoteDataSource
) { ... }

@Module
class NetworkModule {
    // Way to scope types inside a Dagger Module
    @Singleton
    @Provides
    fun provideLoginRetrofitService(): LoginRetrofitService { ... }
}
```
Add scope annotations in classes when using constructor injection (with `@Inject`) and add them in `@Provides` methods when using Dagger modules.

In order to scope elements to the flow you have to use different component with different scope.
If you need dependencies from the main component you can use @Subcomponent.
```kotlin
@ActivityScope
@Subcomponent
interface LoginComponent {

    @Subcomponent.Factory
    interface Factory {
        fun create(): LoginComponent
    }

    // All LoginActivity, LoginUsernameFragment and LoginPasswordFragment
    // request injection from LoginComponent. The graph needs to satisfy
    // all the dependencies of the fields those classes are injecting
    fun inject(loginActivity: LoginActivity)
    fun inject(usernameFragment: LoginUsernameFragment)
    fun inject(passwordFragment: LoginPasswordFragment)
}
```

You have to provide new component to the main component by doint the following:
1. Creating a new Dagger module (e.g. `SubcomponentsModule`) passing the subcomponent's class to the `subcomponents` attribute of the annotation.
```kotlin
// The "subcomponents" attribute in the @Module annotation tells Dagger what
// Subcomponents are children of the Component this module is included in.
@Module(subcomponents = LoginComponent::class)
class SubcomponentsModule {}
```
2. Adding the new module (i.e. `SubcomponentsModule`) to `ApplicationComponent`:
```kotlin
// Including SubcomponentsModule, tell ApplicationComponent that
// LoginComponent is its subcomponent.
@Singleton
@Component(modules = [NetworkModule::class, SubcomponentsModule::class])
interface ApplicationComponent {
}
```
3. Expose the factory that creates instances of `LoginComponent`in the interface:
```kotlin
@Singleton
@Component(modules = [NetworkModule::class, SubcomponentsModule::class])
interface ApplicationComponent {
// This function exposes the LoginComponent Factory out of the graph so consumers
// can use it to obtain new instances of LoginComponent
fun loginComponent(): LoginComponent.Factory
}
```

Then you have to provide this component inside Activity (or navigation destination) that contains all Fragments:
```kotlin
class LoginActivity: Activity() {
    // Reference to the Login graph
    lateinit var loginComponent: LoginComponent

    // Fields that need to be injected by the login graph
    @Inject lateinit var loginViewModel: LoginViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        // Creation of the login graph using the application graph
        loginComponent = (applicationContext as MyDaggerApplication)
                            .appComponent.loginComponent().create()

        // Make Dagger instantiate @Inject fields in LoginActivity
        loginComponent.inject(this)

        // Now loginViewModel is available

        super.onCreate(savedInstanceState)
    }
}
```

You have to extract Module dependency to be declared only once inside component:
```kotlin
// Bad practice: ModuleX is declared multiple times in this Dagger graph
@Component(modules = [Module1::class, Module2::class])
interface ApplicationComponent { ... }

@Module(includes = [ModuleX::class])
class Module1 { ... }

@Module(includes = [ModuleX::class])
class Module2 { ... }
```

```kotlin
//Good practice
@Component(modules = [Module1::class, Module2::class, ModuleX::class])
interface ApplicationComponent { ... }

@Module
class Module1 { ... }

@Module
class Module2 { ... }
```

```kotlin
//Good practice
@Component(modules = [Module1::class, Module2::class, ModuleXCommon::class])
interface ApplicationComponent { ... }

@Module
class ModuleXCommon { ... }

@Module
class ModuleXWithModule1SpecificDependencies { ... }

@Module
class ModuleXWithModule2SpecificDependencies { ... }

@Module(includes = [ModuleXWithModule1SpecificDependencies::class])
class Module1 { ... }

@Module(includes = [ModuleXWithModule2SpecificDependencies::class])
class Module2 { ... }
```

In order to construct object that takes in parameters, which can be known only at runtime Assisted Injection can be used. 

```kotlin
class MyDataService @AssistedInject constructor(
    dataFetcher: DataFetcher,
    @Assisted config: Config
) {}

@AssistedFactory
interface MyDataServiceFactory {
  fun create(config: Config): MyDataService
}

class MyApp {
  @Inject lateinit var serviceFactory: MyDataServiceFactory;

  fun setupService(config: Config): MyDataService {
    val service = serviceFactory.create(config)
    ...
    return service
  }
}
```

- @AssistedInject types cannot be injected directly, only the @AssistedFactory type can be injected. This is true even if the constructor does not contain any assisted parameters.
- @AssistedInject types cannot be scoped.
If there are multiple assisted parameters with the same type qualifier can be used:
```kotlin
class MyDataService @AssistedInject constructor(
    dataFetcher: DataFetcher,
    @Assisted("server") serverConfig: Config,
    @Assisted("client") clientConfig: Config
) {}

@AssistedFactory
interface MyDataServiceFactory {
  fun create(
    @Assisted("server") serverConfig: Config,
    @Assisted("client") clientConfig: Config
  ): MyDataService
}
```

#### Dagger in a multi-module apps
In a multi-module apps there are usually one `app` module, that have a dependency of all other modules and  `base` module, which is a dependency of most of other modules. It's good practice to place `ApplicationComponent` inside `app` module to create graph and provide singleton elements. As an example, classes like `OkHttpClient`, JSON parsers, accessors for your database, or `SharedPreferences` objects that may be defined in the `core` module, will be provided by the `ApplicationComponent` defined in the `app` module.  
Inside other modules we can create subcomponents, if the following cases are true:
- You need to perform field injection, as with `LoginActivityComponent`.
- You need to scope objects, as with `LoginComponent`.
In other cases you would just provide dagger `@Module` `@Provides` or `@Binds` methods if construction injection is not possible for those classes with module specific dependencies (or just `@Inject`, if you have control over dependency) and put it inside `ApplicationComponent`.

Feature modules don't have access to custom Application class because it is located inside `app` module and feature modules cannot be depended on it. That means that they cannot access components initialized inside Application class directly. To implement accessing component we have to define interface and access Application as the implementation of that interface.

```kotlin
interface LoginComponentProvider {
    fun provideLoginComponent(): LoginComponent
}
```

```kotlin
class LoginActivity: Activity() {
  ...

  override fun onCreate(savedInstanceState: Bundle?) {
    loginComponent = (applicationContext as LoginComponentProvider)
                        .provideLoginComponent()

    loginComponent.inject(this)
    ...
  }
}
```

```kotlin
class MyApplication: Application(), LoginComponentProvider {
  // Reference to the application graph that is used across the whole app
  val appComponent = DaggerApplicationComponent.create()

  override fun provideLoginComponent(): LoginComponent {
    return appComponent.loginComponent().create()
  }
}

```

#### Best practices

- The `ApplicationComponent` should always be in the `app` module.
    
- Create Dagger components in modules if you need to perform field injection in that module or you need to scope objects for a specific flow of your application.
    
- For Gradle modules that are meant to be utilities or helpers and don't need to build a graph (that's why you'd need a Dagger component), create and expose public Dagger modules with @Provides and @Binds methods of those classes that don't support constructor injection.
    
- To use Dagger in an Android app with feature modules, use component dependencies to be able to access dependencies provided by the `ApplicationComponent` defined in the `app` module.

### Hilt
Hilt is a DI framework built on top of Dagger. 
All apps that use Hilt must contain an [`Application`](https://developer.android.com/reference/android/app/Application) class that is annotated with `@HiltAndroidApp`.
`@HiltAndroidApp` triggers Hilt's code generation, including a base class for your application that serves as the application-level dependency container.

```kotlin
@HiltAndroidApp
class ExampleApplication : Application() { ... }
```

In order to inject Android classes we have to annotate it with `@AndroidEntryPoint` 
```kotlin
@AndroidEntryPoint
class ExampleActivity : AppCompatActivity() { ... }
```
Hilt currently supports the following Android classes:

- `Application` (by using `@HiltAndroidApp`)
- `ViewModel` (by using `@HiltViewModel`)
- `Activity`
- `Fragment`
- `View`
- `Service`
- `BroadcastReceiver`
`@AndroidEntryPoint` generates an individual Hilt component for each Android class in your project. These components can receive dependencies from their respective parent classes.

A Hilt module is a class that is annotated with `@Module`. Like a [Dagger module](https://developer.android.com/training/dependency-injection/dagger-android#dagger-modules), it informs Hilt how to provide instances of certain types. Unlike Dagger modules, you must annotate Hilt modules with `@InstallIn` to tell Hilt which Android class each module will be used or installed in.

```kotlin
interface AnalyticsService {
  fun analyticsMethods()
}

// Constructor-injected, because Hilt needs to know how to
// provide instances of AnalyticsServiceImpl, too.
class AnalyticsServiceImpl @Inject constructor(
  ...
) : AnalyticsService { ... }

@Module
@InstallIn(ActivityComponent::class)
abstract class AnalyticsModule {

  @Binds
  abstract fun bindAnalyticsService(
    analyticsServiceImpl: AnalyticsServiceImpl
  ): AnalyticsService
}
```

```kotlin
@Module
@InstallIn(ActivityComponent::class)
object AnalyticsModule {

  @Provides
  fun provideAnalyticsService(
    // Potential dependencies of this type
  ): AnalyticsService {
      return Retrofit.Builder()
               .baseUrl("https://example.com")
               .build()
               .create(AnalyticsService::class.java)
  }
}
```

In order to provide different implementations for the same type we can define Qualifier annotation class and annotate classes with it.

```kotlin
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class AuthInterceptorOkHttpClient

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class OtherInterceptorOkHttpClient
```

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

  @AuthInterceptorOkHttpClient
  @Provides
  fun provideAuthInterceptorOkHttpClient(
    authInterceptor: AuthInterceptor
  ): OkHttpClient {
      return OkHttpClient.Builder()
               .addInterceptor(authInterceptor)
               .build()
  }

  @OtherInterceptorOkHttpClient
  @Provides
  fun provideOtherInterceptorOkHttpClient(
    otherInterceptor: OtherInterceptor
  ): OkHttpClient {
      return OkHttpClient.Builder()
               .addInterceptor(otherInterceptor)
               .build()
  }
}

// As a dependency of a constructor-injected class.
class ExampleServiceImpl @Inject constructor(
  @AuthInterceptorOkHttpClient private val okHttpClient: OkHttpClient
) : ...

// At field injection.
@AndroidEntryPoint
class ExampleActivity: AppCompatActivity() {

  @AuthInterceptorOkHttpClient
  @Inject lateinit var okHttpClient: OkHttpClient
}
```

Hilt provides some predefined qualifiers. For example, as you might need the `Context` class from either the application or the activity, Hilt provides the `@ApplicationContext` and `@ActivityContext` qualifiers.

For each supported Android class there are corresponding component that is used for Field injection:

|Hilt component|Injector for|
|---|---|
|`SingletonComponent`|`Application`|
|`ActivityRetainedComponent`|N/A|
|`ViewModelComponent`|`ViewModel`|
|`ActivityComponent`|`Activity`|
|`FragmentComponent`|`Fragment`|
|`ViewComponent`|`View`|
|`ViewWithFragmentComponent`|`View` annotated with `@WithFragmentBindings`|
|`ServiceComponent`|`Service`|

**Note:** Hilt doesn't generate a component for broadcast receivers because Hilt injects broadcast receivers directly from `SingletonComponent`.

There are scopes that come out of the box from Hilt library that used for Android classes:

|Android class|Generated component|Scope|
|---|---|---|
|`Application`|`SingletonComponent`|`@Singleton`|
|`Activity`|`ActivityRetainedComponent`|`@ActivityRetainedScoped`|
|`ViewModel`|`ViewModelComponent`|`@ViewModelScoped`|
|`Activity`|`ActivityComponent`|`@ActivityScoped`|
|`Fragment`|`FragmentComponent`|`@FragmentScoped`|
|`View`|`ViewComponent`|`@ViewScoped`|
|`View` annotated with `@WithFragmentBindings`|`ViewWithFragmentComponent`|`@ViewScoped`|
|`Service`|`ServiceComponent`|`@ServiceScoped`|
A binding's scope must match the scope of the component where it is installed, so in this example you must install `AnalyticsService` in `SingletonComponent` instead of `ActivityComponent`.

```kotlin
// If AnalyticsService is an interface.
@Module
@InstallIn(SingletonComponent::class)
abstract class AnalyticsModule {

  @Singleton
  @Binds
  abstract fun bindAnalyticsService(
    analyticsServiceImpl: AnalyticsServiceImpl
  ): AnalyticsService
}

// If you don't own AnalyticsService.
@Module
@InstallIn(SingletonComponent::class)
object AnalyticsModule {

  @Singleton
  @Provides
  fun provideAnalyticsService(): AnalyticsService {
      return Retrofit.Builder()
               .baseUrl("https://example.com")
               .build()
               .create(AnalyticsService::class.java)
  }
}
```

Installing a module into a component allows its bindings to be accessed as a dependency of other bindings in that component or in any child component below it in the component hierarchy:
![[Pasted image 20240129142900.png]]

In order to field inject dependencies to the classes not directly supported by Hilt `@EntryPoint` annotation can be used:
```kotlin
class ExampleContentProvider : ContentProvider() {

  @EntryPoint
  @InstallIn(SingletonComponent::class)
  interface ExampleContentProviderEntryPoint {
    fun analyticsService(): AnalyticsService
  }

  ...
}
```

To access an entry point, use the appropriate static method from `EntryPointAccessors`. The parameter should be either the component instance or the `@AndroidEntryPoint` object that acts as the component holder. Make sure that the component you pass as a parameter and the `EntryPointAccessors` static method both match the Android class in the `@InstallIn` annotation on the `@EntryPoint` interface:

```kotlin
class ExampleContentProvider: ContentProvider() {
    ...

  override fun query(...): Cursor {
    val appContext = context?.applicationContext ?: throw IllegalStateException()
    val hiltEntryPoint =
      EntryPointAccessors.fromApplication(appContext, ExampleContentProviderEntryPoint::class.java)

    val analyticsService = hiltEntryPoint.analyticsService()
    ...
  }
}
```

![[Pasted image 20240202081122.png]]

## Koin
Koin is a lightweight DI framework based on Kotlin DSL code.
`KoinApplication` (dependency graph) is an instance of koin framework container that contains modules, loggers and properties. We can use `startKoin {}` (usually inside custom application class) to create instance of `KoinApplication` and register it into `GlobalContext` (singleton that holds `KoinApplication`).
To create module we can use `module{}` function. Inside it we can use `single` function for creating singleton object. These objects will be retrieved by Koin container and the same unique instance will be provided. By using `factory` function we tell Koin container not to keep unique instance of class, though new instance of a type will be provided each time it requested: 
```kotlin
val myModule = module {

    // declare Service as single instance
    single { Service() }
    // declare factory instance for Controller class
    factory { Controller(get()) }
}
```

To provide implementation of interface we can use `single<>` function generic type or `as` construction:
```kotlin
// Service interface
interface Service{

    fun doSomething()
}

// Service Implementation
class ServiceImp() : Service {

    fun doSomething() { ... }
}

val myModule = module {

    // Will match type ServiceImp only
    single { ServiceImp() }

    // Will match type Service only
    single<Service> { ServiceImp() }

}
```

If we want to create provide function which provides both implementation and interface we can use `bind` function. This way we can resolve both interface and implementation:
```kotlin
val myModule = module {

    // Will match types ServiceImp & Service
    single { ServiceImp() } bind Service::class
}
```
For different implementations of the same type we can use `named` function to distinguish them:
```kotlin
val myModule = module {
    single<Service>(named("default")) { ServiceImpl() }
    single<Service>(named("test")) { ServiceImpl() }
}

val service : Service by inject(qualifier = named("default"))
```

In order to use parameters (Assisted injection) we can define it inside module and use `by inject()` constuction with `parametersOf()` function:

```kotlin
class Presenter(val view : View)

val myModule = module {
    single{ (view : View) -> Presenter(view) }
}

val presenter : Presenter by inject { parametersOf(view) }


```

`createdAtStart` flag can be used to teel Koin component to create instance or module at the start of `KoinApplication`:
```kotlin
val myModuleA = module {

    single<Service> { ServiceImp() }
}

val myModuleB = module {

    // eager creation for this definition
    single<Service>(createdAtStart=true) { TestServiceImp() }
}
```

```kotlin
val myModuleA = module {

    single<Service> { ServiceImp() }
}

val myModuleB = module(createdAtStart=true) {

    single<Service>{ TestServiceImp() }
}
```

koin doesn't understand difference between two same types but with different generic parameters. So you have to use `named` function to resolve them correctly:
```kotlin
module {
    single(named("Ints")) { ArrayList<Int>() }
    single(named("Strings")) { ArrayList<String>() }
}
```

We can include modules into other modules:
```kotlin
// `:feature` module
val childModule1 = module {
    /* Other definitions here. */
}
val childModule2 = module {
    /* Other definitions here. */
}
val parentModule = module {
    includes(childModule1, childModule2)
}

// `:app` module
startKoin { modules(parentModule) }
```

In order to get instances inside class provided by `KoinApplication` defined inside modules we have to implement KoinComponent interface and then use koin DSL:
```kotlin
class MyComponent : KoinComponent {

    // lazy inject Koin instance
    val myService : MyService by inject()

    // or
    // eager (immediate) inject Koin instance
    val myService : MyService = get()

	// retrieve named instance from given module
	val a = get<ComponentA>(named("A"))
}
```

We can use parameters inside injected components:
```kotlin
class Presenter(val a : A, val b : B)

val myModule = module {
    single { params -> Presenter(a = params.get(), b = params.get()) }
}

class MyComponent : View, KoinComponent {

    val a : A ...
    val b : B ... 

    // inject this as View value
    val presenter : Presenter by inject { parametersOf(a, b) }
}
```

By default in Koin, we have 3 kind of scopes:

- `single` definition, create an object that persistent with the entire container lifetime (can't be dropped).
- `factory` definition, create a new object each time. Short live. No persistence in the container (can't be shared).
- `scoped` definition, create an object that persistent tied to the associated scope lifetime.

To created scoped definition use the following construction: 
```kotlin
module {
    scope<MyType>{
        scoped { Presenter() }
        // ...
    }
}
```

`scope<A> { }` is equivalent to `scope(named<A>()){ }` , but more convenient to write. Note that you can also use a string qualifier like: `scope(named("SCOPE_NAME")) { }`:

```kotlin
class A : KoinScopeComponent {
    override val scope: Scope by lazy { newScope(this) }

    // resolve B as inject
    val b : B by inject() // inject from scope

    // Resolve B
    fun doSomething(){
        val b = get<B>()
    }

    fun close(){
        scope.close() // don't forget to close current scope
    }
}
```

Once your scope instance is finished, just closed it with the `close()` function:

```kotlin
// from a KoinComponent 
val scope = getKoin().createScope<A>()

// use it ...

// close it
scope.close()
```