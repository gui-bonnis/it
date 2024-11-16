Image

## Open & Closed Principle (OCP)
comments /  #solidprinciples #openclosedprinciple #maintainability #flexibility #inheritance #interfaces #polymorphism / @guibonnis / November 16, 2024

> [!Overview]
> The Open/Closed Principle means that a class should be designed so that its behavior can be extended without modifying its existing code. This helps prevent introducing bugs into the existing functionality when adding new features. In Java, this is typically achieved through techniques like inheritance, interfaces, and polymorphism. For example, if we have a base class for payment processing, we can add new payment methods (e.g., credit card, PayPal) by creating new subclasses or implementing an interface, instead of changing the core class. By designing this way, the code is more maintainable, flexible, and adheres to the idea of avoiding direct changes to tested code.

## Behavior Extended
### Creating itself
## Inheritance
## Interfaces
## Polymorphism
## Maintainability
## Flexibility
## Use Cases
### 
### 
## Best Practices
### 
### Design Patterns 
#### Decorator and Strategy Patterns
## Applying OCP in Java
### Using abstraction (interfaces, abstract classes) to allow extensions without modifying the base class
### Implementing design patterns like Strategy, Decorator, and Template Method to adhere to OCP

## Summary:

We were able to successfully demonstrate how to implement the Dependency Injection (DI) design pattern using java and spring framework, also we were able to understand how it states that classes, modules, or functions should be **open for extension but closed for modification** through polymorphism, inheritance and interfaces . Read more about other SOLID Principles.

- [[Single Responsibility Principle (SRP)]]
- [[Liskov Substitution Principle (LSP)]]
- [[Interface Segregation Principle (ISP)]]
- [[Dependency Injection (DI)]]

The source code is available **[here]().**

Happy learning 🙂

---
# Notes

### 3. Open/Closed Principle (OCP)

- Definition of OCP
    - Software entities (classes, modules, functions, etc.) should be open for extension but closed for modification
    - Benefits of OCP: Promotes code reusability and reduces the risk of breaking existing code
- Applying OCP in Java
    - Using abstraction (interfaces, abstract classes) to allow extensions without modifying the base class
    - Implementing design patterns like Strategy, Decorator, and Template Method to adhere to OCP
- Examples of OCP in Java
    - Extending existing classes via inheritance or implementing interfaces
    - Using dependency injection frameworks like Spring to inject behavior without changing existing code
