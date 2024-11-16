Image

## Liskov Substitution Principle (LSP)
comments /  #solidprinciples #liskovsubstitutionprinciple #maintainability #testability #polymorphism #reusability / @guibonnis / November 16, 2024

> [!Overview]
> The Liskov Substitution Principle means that if you have a base class, you should be able to replace it with any of its derived classes without breaking the application. In other words, subclasses should behave in a way that doesn’t violate the expectations set by the base class. This principle ensures that the system remains predictable and adheres to polymorphism.

## Replace Derived Class by Base Class
## Polymorphism
## Reusability
## Use Cases
### 
### 
## Best Practices
### 
### Design Patterns 
#### Adapter Pattern, Factory and Composite Patterns
## Applying LSP in Java
### Ensuring that subclasses maintain the behavior expected from the superclass
### Avoiding changes in the inherited class that would break its expected behavior

## Summary:

We were able to successfully demonstrate how to implement the Liskov Substitution Principle (LSP) design pattern using java, also we were able to understand that LSP focuses on ensuring that derived classes can be used interchangeably with their base classes without altering the correctness of the program. Read more about other SOLID Principles.

- [[Single Responsibility Principle (SRP)]]
- [[Open & Closed Principle (OCP)]]
- [[Interface Segregation Principle (ISP)]]
- [[Dependency Injection (DI)]]

The source code is available **[here]().**

Happy learning 🙂

---
# Notes

### 4. Liskov Substitution Principle (LSP)

- Definition of LSP
    - Objects of a superclass should be replaceable with objects of its subclasses without affecting the correctness of the program
    - Benefits of LSP: Ensures proper inheritance and polymorphism, preventing unexpected behavior in subclasses
- Applying LSP in Java
    - Ensuring that subclasses maintain the behavior expected from the superclass
    - Avoiding changes in the inherited class that would break its expected behavior
- Examples of LSP in Java
    - Correct usage of polymorphism in method overriding
    - Implementing method contracts in subclasses that are consistent with the parent class behavior