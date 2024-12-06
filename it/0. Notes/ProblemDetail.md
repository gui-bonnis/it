If you want to use `org.springframework.http.ProblemDetail` for error handling, you can leverage Spring's built-in support for RFC 7807 (Problem Details for HTTP APIs). Here's how you can do it:

---

### **Step 1: Define Custom Exceptions**

Create specific exceptions for each business error, including error codes.

#### Example: Custom Exceptions

```java
public class BusinessException extends RuntimeException {
    private final String errorCode;

    public BusinessException(String message, String errorCode) {
        super(message);
        this.errorCode = errorCode;
    }

    public String getErrorCode() {
        return errorCode;
    }
}

public class ResourceNotFoundException extends BusinessException {
    public ResourceNotFoundException(String resource, String errorCode) {
        super(resource + " not found", errorCode);
    }
}

public class ValidationException extends BusinessException {
    public ValidationException(String message, String errorCode) {
        super(message, errorCode);
    }
}
```

---

### **Step 2: Configure `@ControllerAdvice`**

Use `@ControllerAdvice` to handle exceptions and generate `ProblemDetail` objects.

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ProblemDetail;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestController;

import java.time.Instant;

@ControllerAdvice
@RestController
public class GlobalExceptionHandler {

    @ExceptionHandler(BusinessException.class)
    public ProblemDetail handleBusinessException(BusinessException ex) {
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, ex.getMessage());
        problemDetail.setTitle("Business Error");
        problemDetail.setProperty("errorCode", ex.getErrorCode());
        problemDetail.setProperty("timestamp", Instant.now());
        return problemDetail;
    }

    @ExceptionHandler(ResourceNotFoundException.class)
    public ProblemDetail handleResourceNotFoundException(ResourceNotFoundException ex) {
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
        problemDetail.setTitle("Resource Not Found");
        problemDetail.setProperty("errorCode", ex.getErrorCode());
        problemDetail.setProperty("timestamp", Instant.now());
        return problemDetail;
    }
}
```

---

### **Step 3: Use Feign Error Decoder for External APIs**

If you're using Feign for external API calls, configure a custom error decoder to map Feign exceptions to your custom exceptions.

```java
import feign.Response;
import feign.codec.ErrorDecoder;
import org.springframework.http.HttpStatus;

public class FeignErrorDecoder implements ErrorDecoder {
    @Override
    public Exception decode(String methodKey, Response response) {
        HttpStatus status = HttpStatus.valueOf(response.status());

        if (status == HttpStatus.NOT_FOUND) {
            return new ResourceNotFoundException("External Resource", "ERR-404");
        } else if (status == HttpStatus.BAD_REQUEST) {
            return new ValidationException("Invalid request to external service", "ERR-400");
        }
        return new BusinessException("Unexpected error from external service", "ERR-500");
    }
}
```

---

### **Step 4: Configure Feign Client**

Register the custom error decoder in your Feign client configuration.

```java
import org.springframework.context.annotation.Bean;
import feign.codec.ErrorDecoder;

public class FeignClientConfig {
    @Bean
    public ErrorDecoder errorDecoder() {
        return new FeignErrorDecoder();
    }
}
```

---

### **Example Scenario**

#### Custom Exception Thrown

If a resource is not found:

```java
throw new ResourceNotFoundException("Order ID 12345", "ERR-404");
```

#### Response Output

```json
{
    "title": "Resource Not Found",
    "status": 404,
    "detail": "Order ID 12345 not found",
    "errorCode": "ERR-404",
    "timestamp": "2024-12-02T12:34:56Z"
}
```

---

### **Benefits**

1. **Standardized Error Format**: Using `ProblemDetail` ensures consistency in API responses.
2. **Customizability**: Add business-specific properties to the error response.
3. **Integration with Feign**: Handle external API errors gracefully.

This setup provides a robust and standardized way to handle exceptions in a microservice using Spring Boot.


The `type` field in a `ProblemDetail` object should contain a URI that identifies the type of problem encountered. The URI can be an absolute or relative link and typically points to a resource (like documentation) describing the error type in detail. If no such documentation exists, you can use a placeholder URI or a meaningful string identifier.

---

### **Options for `type`**

1. **Link to Documentation**
    
    - Use a URL pointing to documentation about the specific error.
    - Example: `https://example.com/docs/errors/resource-not-found`
2. **Custom Identifier**
    
    - If you don’t have documentation, use a meaningful string or a pseudo-URI.
    - Example: `urn:example:errors:resource-not-found`
3. **Generic Identifiers**
    
    - Use a generic type that classifies the problem.
    - Example:
        - `https://example.com/docs/errors/validation-error`
        - `urn:example:errors:internal-server-error`

---

### **Best Practices**

- **Consistency**: Use a consistent pattern for all your problem types, such as `https://your-domain.com/docs/errors/<type>` or `urn:your-app:errors:<type>`.
- **Traceability**: Make sure the `type` gives enough context to identify the error uniquely.
- **Readability**: Avoid overly complex URIs; make them self-explanatory.

---

### **Example Implementation**

Here’s how you might use the `type` field in your code:

```java
@ExceptionHandler(ResourceNotFoundException.class)
public ProblemDetail handleResourceNotFoundException(ResourceNotFoundException ex) {
    ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
    problemDetail.setTitle("Resource Not Found");
    problemDetail.setType("https://example.com/docs/errors/resource-not-found");
    problemDetail.setProperty("errorCode", ex.getErrorCode());
    problemDetail.setProperty("timestamp", Instant.now());
    return problemDetail;
}
```

#### Response Example:

```json
{
    "type": "https://example.com/docs/errors/resource-not-found",
    "title": "Resource Not Found",
    "status": 404,
    "detail": "Order ID 12345 not found",
    "errorCode": "ERR-404",
    "timestamp": "2024-12-02T12:34:56Z"
}
```

This provides clear and actionable information, helping developers and users trace the issue effectively.


The `org.springframework.http.ProblemDetail` class is designed to comply with [RFC 7807](https://www.rfc-editor.org/rfc/rfc7807), which defines a standard format for HTTP API error responses. The class includes predefined properties and allows additional custom properties to be added. Here’s a breakdown of the properties available:

---

### **Standard Properties**

1. **`type` (String)**
    
    - A URI reference that identifies the problem type.
    - Default: `about:blank` if not explicitly set.
    - Example: `https://example.com/errors/resource-not-found`
2. **`title` (String)**
    
    - A short, human-readable summary of the problem type.
    - Default: Corresponds to the HTTP status if not explicitly set.
    - Example: `"Resource Not Found"`
3. **`status` (Integer)**
    
    - The HTTP status code for the problem.
    - Example: `404`
4. **`detail` (String)**
    
    - A human-readable explanation of the problem, specific to the occurrence.
    - Example: `"The order with ID 12345 was not found"`
5. **`instance` (String)**
    
    - A URI reference that identifies the specific occurrence of the problem.
    - Example: `"/orders/12345"`

---

### **Custom Properties**

You can add custom properties to `ProblemDetail` to include additional information. These properties are added using the `setProperty` method.

#### Examples of Custom Properties:

1. **`errorCode` (String)**
    
    - A business-specific error code.
    - Example: `"ERR-404"`
2. **`timestamp` (String)**
    
    - The time the error occurred, typically in ISO 8601 format.
    - Example: `"2024-12-02T12:34:56Z"`
3. **`fieldErrors` (Array)**
    
    - For validation errors, a list of fields with issues.
    - Example:
        
        ```json
        [
            { "field": "email", "message": "Email format is invalid" },
            { "field": "password", "message": "Password must be at least 8 characters long" }
        ]
        ```
        
4. **`serviceName` (String)**
    
    - Identifies the service where the error occurred.
    - Example: `"OrderService"`

---

### **Adding Custom Properties**

You can set custom properties dynamically using the `setProperty` method.

#### Example Code:

```java
@ExceptionHandler(ValidationException.class)
public ProblemDetail handleValidationException(ValidationException ex) {
    ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, ex.getMessage());
    problemDetail.setTitle("Validation Error");
    problemDetail.setType("https://example.com/errors/validation-error");
    problemDetail.setProperty("errorCode", ex.getErrorCode());
    problemDetail.setProperty("timestamp", Instant.now());
    problemDetail.setProperty("fieldErrors", ex.getFieldErrors());
    return problemDetail;
}
```

#### Response Example:

```json
{
    "type": "https://example.com/errors/validation-error",
    "title": "Validation Error",
    "status": 400,
    "detail": "Invalid request data",
    "errorCode": "ERR-400",
    "timestamp": "2024-12-02T12:34:56Z",
    "fieldErrors": [
        { "field": "email", "message": "Email format is invalid" },
        { "field": "password", "message": "Password must be at least 8 characters long" }
    ]
}
```

---

### **Default Methods for Property Access**

- **`getProperty(String name)`**: Retrieves the value of a custom property.
- **`setProperty(String name, Object value)`**: Adds a custom property.
- **`getProperties()`**: Retrieves all custom properties as a `Map`.

---

### **Flexibility**

The combination of standard and custom properties makes `ProblemDetail` highly adaptable for various use cases, allowing you to provide meaningful error information while maintaining compliance with HTTP standards.


Here’s an example of using `ProblemDetail` in a reactive Spring Boot application with `@ControllerAdvice`, handling exceptions and returning a `Mono<ProblemDetail>` as the error response.

---

### **Scenario**

The application handles a reactive API where:
- A `ResourceNotFoundException` is thrown if a resource is not found.
- A `ValidationException` is thrown for invalid input.

---

### **Code Implementation**

#### **1. Define Custom Exceptions**

```java
public class BusinessException extends RuntimeException {
    private final String errorCode;

    public BusinessException(String message, String errorCode) {
        super(message);
        this.errorCode = errorCode;
    }

    public String getErrorCode() {
        return errorCode;
    }
}

public class ResourceNotFoundException extends BusinessException {
    public ResourceNotFoundException(String resourceId) {
        super("Resource with ID " + resourceId + " not found", "ERR-404");
    }
}

public class ValidationException extends BusinessException {
    public ValidationException(String message) {
        super(message, "ERR-400");
    }
}
```

---

#### **2. Create a Global Exception Handler**

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ProblemDetail;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import reactor.core.publisher.Mono;

import java.time.Instant;

@ControllerAdvice
public class ReactiveExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public Mono<ProblemDetail> handleResourceNotFoundException(ResourceNotFoundException ex) {
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
        problemDetail.setTitle("Resource Not Found");
        problemDetail.setType("https://example.com/errors/resource-not-found");
        problemDetail.setProperty("errorCode", ex.getErrorCode());
        problemDetail.setProperty("timestamp", Instant.now());
        return Mono.just(problemDetail);
    }

    @ExceptionHandler(ValidationException.class)
    public Mono<ProblemDetail> handleValidationException(ValidationException ex) {
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, ex.getMessage());
        problemDetail.setTitle("Validation Error");
        problemDetail.setType("https://example.com/errors/validation-error");
        problemDetail.setProperty("errorCode", ex.getErrorCode());
        problemDetail.setProperty("timestamp", Instant.now());
        return Mono.just(problemDetail);
    }

    @ExceptionHandler(Exception.class)
    public Mono<ProblemDetail> handleGenericException(Exception ex) {
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(HttpStatus.INTERNAL_SERVER_ERROR, "An unexpected error occurred");
        problemDetail.setTitle("Internal Server Error");
        problemDetail.setType("https://example.com/errors/internal-server-error");
        problemDetail.setProperty("timestamp", Instant.now());
        return Mono.just(problemDetail);
    }
}
```

---

#### **3. Example Controller**

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Mono;

@RestController
public class ReactiveController {

    @GetMapping("/resource/{id}")
    public Mono<String> getResource(@PathVariable String id) {
        if ("404".equals(id)) {
            throw new ResourceNotFoundException(id);
        } else if ("400".equals(id)) {
            throw new ValidationException("Invalid resource ID");
        }
        return Mono.just("Resource ID: " + id);
    }
}
```

---

#### **4. Example Responses**

**Request 1:**  
`GET /resource/404`

**Response:**
```json
{
    "type": "https://example.com/errors/resource-not-found",
    "title": "Resource Not Found",
    "status": 404,
    "detail": "Resource with ID 404 not found",
    "errorCode": "ERR-404",
    "timestamp": "2024-12-02T12:34:56Z"
}
```

**Request 2:**  
`GET /resource/400`

**Response:**
```json
{
    "type": "https://example.com/errors/validation-error",
    "title": "Validation Error",
    "status": 400,
    "detail": "Invalid resource ID",
    "errorCode": "ERR-400",
    "timestamp": "2024-12-02T12:34:56Z"
}
```

**Request 3:**  
`GET /resource/500`

**Response:**
```json
{
    "type": "https://example.com/errors/internal-server-error",
    "title": "Internal Server Error",
    "status": 500,
    "detail": "An unexpected error occurred",
    "timestamp": "2024-12-02T12:34:56Z"
}
```

---

### **Key Points**

1. **Reactive Response**:  
   `Mono<ProblemDetail>` is used to return the error response in a reactive manner.

2. **Standardized Error Format**:  
   The use of `ProblemDetail` ensures consistent error responses across the application.

3. **Customizable Details**:  
   Additional properties (`errorCode`, `timestamp`, etc.) provide context for debugging and user guidance.

4. **Generic Fallback**:  
   The generic `Exception` handler acts as a safety net for unexpected errors.

This setup ensures a reactive and standardized error-handling mechanism in a Spring WebFlux application.