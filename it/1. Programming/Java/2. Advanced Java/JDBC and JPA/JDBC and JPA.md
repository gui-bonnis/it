Here’s a detailed list of topics for **Advanced Java > JDBC and JPA**:

## 1. **Introduction to JDBC and JPA**
    
    - Overview of JDBC and JPA
    - Differences Between JDBC and JPA
    - When to Use JDBC vs. JPA

## 2. **JDBC Basics**
    
    - Setting Up JDBC Drivers and Connections
    - CRUD Operations with JDBC
    - Executing SQL Statements (`Statement`, `PreparedStatement`, `CallableStatement`)
    - Managing Database Connections (Pooling, Closing)

## 3. **Advanced JDBC Features**
    
    - Transaction Management with JDBC (`commit`, `rollback`)
    - Batch Processing and Bulk Operations
    - Working with Large Objects (LOBs, BLOBs, CLOBs)
    - Result Set Handling (`ResultSet` Types and Scrolling)
    - Metadata (`DatabaseMetaData`, `ResultSetMetaData`)

## 4. **Error Handling and Best Practices in JDBC**
    
    - Handling SQL Exceptions
    - Proper Use of Try-with-Resources
    - Performance Optimization (Batching, Indexing)
    - Preventing SQL Injection with `PreparedStatement`

## 5. **Introduction to JPA**
    
    - JPA Architecture and Key Concepts
    - Setting Up JPA (Persistence Unit, `persistence.xml`)
    - Working with JPA Providers (Hibernate, EclipseLink)

## 6. **Entity Mapping in JPA**
    
    - Mapping Entities to Database Tables (`@Entity`, `@Table`)
    - Field and Column Mappings (`@Column`, `@Id`, `@GeneratedValue`)
    - Embeddable Types (`@Embedded`, `@Embeddable`)
    - Inheritance Mapping Strategies (`@Inheritance`, `@MappedSuperclass`)

## 7. **Relationships in JPA**
    
    - One-to-One, One-to-Many, Many-to-One, and Many-to-Many Mappings
    - `@JoinColumn`, `@JoinTable`, and `@MappedBy`
    - Cascading Operations (`CascadeType`)
    - Lazy vs. Eager Loading

## 8. **JPA Querying with JPQL and Criteria API**
    
    - JPQL Basics and Syntax
    - Named Queries (`@NamedQuery`, `@NamedNativeQuery`)
    - Criteria API for Type-Safe Queries
    - Pagination and Sorting in JPA Queries

## 9. **Transaction Management in JPA**
    
    - JPA Transactions and `EntityTransaction`
    - Using `@Transactional` in JPA
    - Optimistic and Pessimistic Locking (`@Version`, Lock Modes)

## 10. **Advanced JPA Features**
    
    - Lifecycle Callbacks (`@PrePersist`, `@PostPersist`, etc.)
    - Entity Listeners for Auditing and Logging
    - Working with Native Queries
    - Database Schema Generation

## 11. **Caching in JPA**
    
    - First-Level Cache (Persistence Context)
    - Second-Level Cache (`@Cacheable`)
    - Cache Providers and Configuration
    - Managing Cache Consistency and Invalidation

## 12. **JPA Best Practices and Performance Tuning**
    
    - Efficient Use of Fetch Types (Lazy vs. Eager)
    - Minimizing N+1 Select Problem
    - Batch Fetching and Fetch Joins
    - Avoiding Memory Leaks with Detached Entities
