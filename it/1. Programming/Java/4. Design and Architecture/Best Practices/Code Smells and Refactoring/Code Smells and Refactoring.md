## Code Smells and Refactoring

- Identifying Code Smells
    - Long Methods
    - Large Classes
    - Duplicate Code
    - Inconsistent Naming
    - Feature Envy (When one class frequently accesses another class’s methods)
    - Data Clumps (Groups of data items that often go together but aren’t encapsulated in an object)
    - Switch Statements (Frequent use of `switch` or `if-else` blocks, suggesting poor design)
    - God Objects (Objects that have too many responsibilities)
- Refactoring Techniques
    - Extract Method, Extract Class
    - Rename Method, Rename Class
    - Replace Conditional with Polymorphism
    - Move Method, Move Field
    - Simplifying Complex Conditionals
    - Dealing with Long Parameter Lists
    - Encapsulate Field
    - Using the Strategy Pattern to refactor conditional logic
- Test-Driven Refactoring
    - Ensuring refactoring does not break existing functionality
    - Running unit tests before and after refactoring
- Refactoring for Performance
    - Optimizing loops and recursive methods
    - Minimizing unnecessary object creation
    - Reducing complexity to improve maintainability and readability