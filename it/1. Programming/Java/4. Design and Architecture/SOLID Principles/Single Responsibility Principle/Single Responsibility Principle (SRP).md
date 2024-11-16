Image

## Single Responsibility Principle (SRP)
comments /  #solidprinciples #singleresponsibilityprinciple #maintainability #testability / @guibonnis / November 16, 2024

> [!Overview]
> The Single Responsibility Principle means that a class should be responsible for only one specific part of the functionality in an application. It should focus on doing one thing well and have only one reason to change. By adhering to SRP, we make our code easier to understand, test, and maintain. For example, in a Java application, instead of having a single class that handles both file operations and data processing, we would separate these concerns into two classes: one for managing file operations and another for processing data. This separation ensures that changes to file handling won't affect data processing logic, and vice versa.

## Focus on One Thing
### Have only one reason to change
## Easy to understand
## Easy test
## Easy maintain
### Improve modularity
## Use Cases
### 
### 
### 
## Best Practices
### 
### Design Patterns 
#### Proxy, Command, and Strategy Patterns
## Applying SRP in Java
### Identifying classes that violate SRP
### Refactoring code to ensure each class has a single responsibility

## Summary

We were able to successfully demonstrate how to implement the Single Responsibility Principle (SRP) design pattern using java, also we were able to understand that a class should have only one reason to change, meaning it should focus on a single responsibility or functionality. Read more about other SOLID Principles.

- [[Open & Closed Principle (OCP)]]
- [[Liskov Substitution Principle (LSP)]]
- [[Interface Segregation Principle (ISP)]]
- [[Dependency Injection (DI)]]
 

The source code is available **[here]().**

Happy learning 🙂

---
# Notes

### **2. Single Responsibility Principle (SRP)**

- **Definition of SRP**
    - A class should have only one reason to change, meaning it should have only one job or responsibility
    - Benefits of SRP: Easier testing, understanding, and maintaining code
- **Applying SRP in Java**
    - Identifying classes that violate SRP
    - Refactoring code to ensure each class has a single responsibility
- **Examples of SRP in Java**
    - Designing classes with focused responsibilities, such as separating business logic and data access code