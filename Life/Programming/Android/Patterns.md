## Factory Method
Factory method is used to decouple object creation(instantiation) from client logic of usage of this product. We need to create factory for products that implement the same interface. Inside the factory we have fabric creation method that returns that general inteface. Then we create concrete factory that returns concrete products. To use this patter we need to create logic that depends on abstract products. And use factories. We work with general  factory and then pass concrete factory. We need to use factory creation methods, instead of just using object instantiation, in order to change creation of an object from one place (from inside factory method) and not to change every object creation through all the application.
## Abstract Factory
Abstract factory creates family of the same-type products using general interface of those products. It decouples products creation from usage of products and makes it easy to maintain such code. If we already have factory method implemented in code, and after refactoring we need to add more classes, we may not create new factory for this class, but move these classes in single AbstractFactory. It is used in cases when we need to provide a bunch of same-type classes that need to work all together. Or when you want to provide access to components of your library without giving out implementation details.
## Builder
Builder is a patterns which is used for creation of complex objects with a lot of fields and nested classes, especially with optional fields. You need to create abstract builder class that defines each existing initialization operation, then concrete builder class defines product that gets created and method that return complete product and overrides initialization methods. There is also optional director role, that can define concrete steps of building product. 
## Prototype
Prototype is a pattern which allows to create clone of the other object instance based on it without the need to know all the details of the inner structure, but only using clone() method. It is used when we want to create a different version of the class, we may not use constructor, but create it based on the existing one, so we just clone the prototype instance and change the field we want. Thus, we avoid creating complex hierarchy of objects inside constructor and recreating objects that would be the same.
A real-world example of the Prototype Design Pattern is the creation of a web form builder. Consider a scenario where a user wants to create a form with multiple fields, such as text fields, drop-down lists, and checkboxes. Instead of creating a new instance of each field type from scratch, the user can use the Prototype Design Pattern to clone existing field prototypes and modify the properties of the cloned fields as needed.

Here's how it could work in practice:

1.  A prototype class is created for each field type, such as TextFieldPrototype, DropdownListPrototype, and CheckboxPrototype.
    
2.  The prototype classes implement the "clone" method to create a copy of the object.
    
3.  A factory class is created to manage the prototypes and provide the client code with a clone of the desired prototype.
    
4.  The client code requests a clone of the desired prototype from the factory class and modifies the properties of the cloned object, such as the label, placeholder, and default value.
    
5.  The client code repeats the process to create additional fields and add them to the form.
    

By using the Prototype Design Pattern, the client code can create new fields quickly and efficiently by cloning existing prototypes, rather than creating each field from scratch. This can save time and reduce the complexity of the code.