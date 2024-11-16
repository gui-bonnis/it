# SOLID Principles
comments /  #solidprinciples #designpatterns #singleresponsibilityprinciple #openclosedprinciple #liskovsubstitutionprinciple #interfacesegregationprinciple #dependencyinjection  / @guibonnis / November 16, 2024

## What is SOLID

### Definition and overview of SOLID principles in Object-Oriented Programming (OOP)

### How SOLID principles improve software maintainability, scalability, and readability

### History and origin of SOLID principles (coined by Robert C. Martin)

![[Single Responsibility Principle (SRP)#Single Responsibility Principle (SRP)]]

![[Open & Closed Principle (OCP)#Open & Closed Principle (OCP)]]

![[Liskov Substitution Principle (LSP)#Liskov Substitution Principle (LSP)]]

![[Interface Segregation Principle (ISP)#Interface Segregation Principle (ISP)]]

![[Dependency Injection (DI)#Dependency Injection (DI)]]


## SOLID Design Patterns

- Patterns Related to SOLID Principles
    - Strategy Pattern: Facilitates OCP by allowing the definition of algorithms in separate classes that can be easily extended.
    - Factory Method Pattern: Facilitates DIP by providing a way to delegate object creation to subclasses without depending on concrete classes.
    - Decorator Pattern: Facilitates OCP by allowing dynamic behavior changes for classes at runtime.
    - Adapter Pattern: Facilitates LSP by allowing the substitution of classes with different interfaces while maintaining compatibility.
    - Composite Pattern: Facilitates ISP by allowing the composition of smaller interfaces in complex structures.

## Refactoring for SOLID Principles

- Identifying Code Smells in the context of SOLID
    - Detecting and addressing violations of SRP, OCP, LSP, ISP, and DIP in legacy code
- Techniques for Refactoring to conform to SOLID principles
    - Using Extract Method, Extract Interface, and Rename Class refactorings to improve class responsibilities
    - Breaking down large classes, methods, and interfaces to ensure adherence to SRP, ISP, and other principles
- Example Refactorings
    - Refactoring large, monolithic classes into smaller classes adhering to SRP
    - Implementing OCP by introducing interfaces or abstract classes to extend behavior without modifying the core classes

## Benefits of Following SOLID Principles

- Improved Code Quality
    - Code that is easier to maintain, extend, and test
    - Better adherence to object-oriented design principles
- Scalability and Flexibility
    - Systems are more flexible to changes and additions, reducing risk when new features are introduced
- Testability
    - Promotes writing unit tests for each class, ensuring that small, focused classes can be tested independently

## Challenges in Applying SOLID Principles

- Balancing SOLID with Pragmatism
    - Knowing when to apply SOLID and when not to overcomplicate simple systems
    - Striking a balance between adhering to SOLID principles and keeping the design simple and understandable
- Learning Curve and Complexity
    - Initial challenges when adopting SOLID principles in existing systems
    - Understanding the deep integration of design patterns and principles to achieve SOLID designs

## Advanced Concepts Related to SOLID Principles

- Design by Contract (DbC)
    - Leveraging preconditions, postconditions, and invariants to ensure LSP and OCP are followed
- Composition over Inheritance
    - Encouraging the use of composition to favor OCP and avoid Liskov violations due to complex inheritance hierarchies


## Summary

Read more about other SOLID Principles

- [[Single Responsibility Principle (SRP)]]
- [[Open & Closed Principle (OCP)]]
- [[Liskov Substitution Principle (LSP)]]
- [[Interface Segregation Principle (ISP)]]
- [[Dependency Injection (DI)]]

Happy learning 🙂

---
# Notes