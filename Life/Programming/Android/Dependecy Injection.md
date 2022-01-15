Dependency injection is a process of supplying dependencies to client that uses them. There are dependencies, client and injector. Injector is responsible for supplying all needed dependencies to client through, usually, factories or builders. The point of this is to allow separation of concern, which means that every section of program is responsible for different logic. This allows code reuse, easy testing and more code readability. 
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