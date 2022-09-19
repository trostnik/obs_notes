Dependency injection is a process of supplying dependencies to client that uses them. The point is client doesn't know how he gets dependencies, it's done from the outside. There are dependencies, client and injector. Injector is responsible for supplying all needed dependencies to client through, usually, factories or builders. The point of this is to allow separation of concern, which means that every section of program is responsible for different logic. This allows code reuse, easy testing and more code readability. 
Service Locator is a registry that contains actual instances and a way to get them.
# Dagger
Dagger 2 is a library which allows to implement dependency injection through annotation processing without much boilerplate code. To use dagger we need to use these annotations:
1. @Inject it is an android annotation that tells that the class can be a part of dependency graph and is also need to be provided with it's own dependencies. When used on field, dagger injects this field if a client is a part of a graph. When used on method this method supposed to be called only once from dagger itself.
2. @Module is an annotation that tells dagger that class will provide dependencies. Inside module class you need to provide dependencies that are either a part of library, which means you don't have access to creating objects and you can create them only though library's api, or dependencies that are provided through interfaces
3. @Provides is an annotation that tells that the method provides certain dependency  
4. @Singleton is used as a scope, it tells dagger that dependency is an one instance dependency and it needs to provide it to all of clients. We need to annotate component and dependency with this annotation and use the same component, which will store the instance of dependency. We can also create our own scopes.
5. @Binds is used for interfaces and generate way less code. It also allows to escape manual creation of object inside @Provides method 
6. @Component -  is an annotation that constructs dependency graph and connects clients with dependencies. It is provided with module and usually have method inject() which takes in client class. The component class needs to be accessible from all the clients and this is done through creating application class which runs before any Android components when process is started.
7. @Named is used to differentiate dependencies when they have the same type
8. @Component.Builder, @Component.Factory it is used for parametrized dependency(when we need to provide some dependency to a module on creation)
	![[Pasted image 20220115133649.png]]
	![[Pasted image 20220115133607.png]]
	![[Pasted image 20220115133703.png]]
	![[Pasted image 20220115133710.png]]

# Hilt
Hilt is a dependency injection library built on top of dagger which allows to avoid boilerplate code of constructing classes and it's dependencies by providing containers for every Android class and managin it's lifecycle automatically.

@HiltAndroidApp - is an annotation that used for Application class and it's required for every application that uses Hilt. It triggers Hilt code generation.

@AndroidEntryPoint - is an annotation that is used on clients that want to consume dependencies
Hilt currently supports the following Android classes:
	Application (by using @HiltAndroidApp)
	ViewModel (by using @HiltViewModel)
	Activity
	Fragment
	View
	Service
	BroadcastReceiver

If you annotate an Android class with @AndroidEntryPoint, then you also must annotate Android classes that depend on it. For example, if you annotate a fragment, then you must also annotate any activities where you use that fragment.

We can use modules but, unlike Dagger @Module annotation, in Hilt we need to use @InstallIn annotation and provide it with Android component.
For multiple bindings(when you need to provide different implementations of the same type) @Qualifier is used:
```kotlin
@Qualifier  
@Retention(AnnotationRetention.BINARY)  
annotation class AuthInterceptorOkHttpClient  
  
@Qualifier  
@Retention(AnnotationRetention.BINARY)  
annotation class OtherInterceptorOkHttpClient
```

then you just annotate the field or parameter with annotation class name.

Hilt provides some predefined qualifiers. For example, as you might need the Context class from either the application or the activity, Hilt provides the @ApplicationContext and @ActivityContext qualifiers.

Hilt provides the following components:
![[Pasted image 20220130141709.png]]
ActivityRetainedComponent lives across configuration changes, so it is created at the first Activity#onCreate() and destroyed at the last Activity#onDestroy()
 
The table below lists scope annotations for each generated component:
![[Pasted image 20220130142136.png]]

 If there is a need  to use Hilt in non-Android classes to get some dependencies, we need to define an interface that is annotated with @EntryPoint for each binding type that you want and include qualifiers. Then add @InstallIn to specify the component in which to install the entry point as follows:
 
 ``` kotlin
 class ExampleContentProvider : ContentProvider() {  
	 @EntryPoint  
	 @InstallIn(SingletonComponent::class)  interface ExampleContentProviderEntryPoint {  fun analyticsService(): AnalyticsService  }  ...  
}
```
To access an entry point, use the appropriate static method from EntryPointAccessors. The parameter should be either the component instance or the @AndroidEntryPoint object that acts as the component holder. Make sure that the component you pass as a parameter and the EntryPointAccessors static method both match the Android class in the @InstallIn annotation on the @EntryPoint interface:
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
}```

In this example, you must use the ApplicationContext to retrieve the entry point because the entry point is installed in SingletonComponent. If the binding that you wanted to retrieve were in the ActivityComponent, you would instead use the ActivityContext.
