Image

## Dependency Injection (DI)
comments /  #solidprinciples #dependencyinjection #maintainability #testability #loosecoupling / @guibonnis / November 16, 2024

> [!Overview]
> Dependency Injection is a pattern where an **object receives its dependencies from an external source** rather than creating them itself. This approach **promotes loose coupling** because the object is no longer responsible for instantiating its dependencies, **allowing us to easily swap or mock dependencies**, which is especially helpful for testing. In Java, DI can be implemented in various ways, such as through constructor injection, setter injection, or field injection. **Frameworks like Spring** provide extensive support for DI, making it easy to configure dependencies in a centralized way, **typically using annotations like** `@Autowired` or configuration files.

## External Source
### Creating itself
## Loose Coupling
## Easy Swap
## Mock dependencies
## Implementation
### Constructor Injection:
### Set Injection
### Field Injection
## Framework Implementation
## Use Cases
### 
### 
## Best Practices
### 
### Design Patterns 
#### Factory Method Pattern, Dependency Injection Pattern
## Applying DIP in Java
### Using interfaces or abstract classes to decouple high-level and low-level modules
### Inversion of control (IoC) through frameworks like Spring to manage dependencies and decouple code
### Constructor injection and setter injection for dependency management

## Summary:

We were able to successfully demonstrate how to implement the Dependency Injection (DI) design pattern using java and spring framework, also we were able to understand how DI make code maintainability and testability. Read more about other SOLID Principles.

- [[Single Responsibility Principle (SRP)]]
- [[Open & Closed Principle (OCP)]]
- [[Liskov Substitution Principle (LSP)]]
- [[Interface Segregation Principle (ISP)]]

The source code is available **[here]().**

Happy learning 🙂

---
# Notes

### 6. Dependency Inversion Principle (DIP)

- Definition of DIP
    - High-level modules should not depend on low-level modules. Both should depend on abstractions (interfaces or abstract classes)
    - Abstractions should not depend on details. Details should depend on abstractions
    - Benefits of DIP: Reduces coupling, promotes code flexibility, and improves testability
- Applying DIP in Java
    - Using interfaces or abstract classes to decouple high-level and low-level modules
    - Inversion of control (IoC) through frameworks like Spring to manage dependencies and decouple code
    - Constructor injection and setter injection for dependency management
- Examples of DIP in Java
    - Designing a service class that depends on an interface rather than a concrete implementation
    - Using the Spring IoC container for automatic dependency injection
    - Avoiding tight coupling between components by depending on abstractions rather than concrete classes