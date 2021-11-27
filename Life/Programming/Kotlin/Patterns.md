Factory Method
Factory method is used to decouple object creation(instantiation) from client logic of usage of this product. We need to create factory for products that implement the same interface. Inside the factory we have fabric creation method that returns that general inteface. Then we create concrete factory that returns concrete products. To use this patter we need to create logic that depends on abstract products. And use factories. We work with general  factory and then pass concrete factory.
Abstract Factory
