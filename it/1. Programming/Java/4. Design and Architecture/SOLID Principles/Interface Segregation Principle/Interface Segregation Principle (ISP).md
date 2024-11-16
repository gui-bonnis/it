Image

## Interface Segregation Principle (ISP)
comments /  #solidprinciples #interfacesegregationprinciple #maintainability #testability #modularity / @guibonnis / November 16, 2024

> [!Overview]
> The Interface Segregation Principle means that an interface should have only the methods that are relevant to the specific implementing class. It encourages creating smaller, more focused interfaces rather than having one large interface with unrelated methods. By following ISP, we ensure that classes only depend on what they actually use, making the code more modular and easier to maintain.

## Specific Implementation
### Smaller X Large Interfaces
## Easy to understand
## Easy test
## Easy maintain
### Improve modularity
## Use Cases
### 
### 
## Best Practices
### 
### Design Patterns 
#### Composite Pattern, Proxy and Adapter Patterns
## Applying ISP in Java
### Creating multiple specific interfaces instead of one large, general-purpose interface
### Using default methods in interfaces (Java 8) to provide default functionality while keeping interfaces small and cohesive

## Summary:

We were able to successfully demonstrate how to implement the Interface Segregation Principle (ISP) design pattern using java, also we were able to understand how ISP focuses on designing interfaces in a way that they are **specific to the client’s needs**. It states that **no** client should be forced to implement methods it doesn’t use. Read more about other SOLID Principles.

- [[Single Responsibility Principle (SRP)]]
- [[Open & Closed Principle (OCP)]]
- [[Liskov Substitution Principle (LSP)]]
- [[Dependency Injection (DI)]]

The source code is available **[here]().**

Happy learning 🙂

---
# Notes

### 5. Interface Segregation Principle (ISP)

- Definition of ISP
    - Clients should not be forced to depend on interfaces they do not use
    - Benefits of ISP: Promotes smaller, more focused interfaces and avoids bloated, monolithic interfaces
- Applying ISP in Java
    - Creating multiple specific interfaces instead of one large, general-purpose interface
    - Using default methods in interfaces (Java 8) to provide default functionality while keeping interfaces small and cohesive
- Examples of ISP in Java
    - Designing several interfaces for different user roles or use cases (e.g., `Readable`, `Writable` interfaces)
    - Refactoring large interfaces into multiple smaller interfaces that represent different responsibilities
