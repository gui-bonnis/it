To connect a `ProblemDetail` response with a list of validation errors using Java and Spring Boot, you can create a custom `ValidationException` and use a `@ControllerAdvice` to handle the exception and generate a structured response.

Here’s an example implementation:

### 1. **Custom ValidationException**

This exception will hold a list of validation errors.

```java
import java.util.List;

public class ValidationException extends RuntimeException {
    private final List<String> errors;

    public ValidationException(List<String> errors) {
        super("Validation errors occurred");
        this.errors = errors;
    }

    public List<String> getErrors() {
        return errors;
    }
}
```

---

### 2. **Service Logic**

Build your list of validation errors using predicates, throw the exception if there are any errors.

```java
import java.util.List;
import java.util.Objects;
import java.util.stream.Stream;

public class PessoaService {

    public void validatePessoa(Pessoa pessoa) {
        List<String> validationErrors = Stream.of(
                pessoaExists.test(pessoa) ? "Pessoa already exists" : null,
                pessoaIsOld.test(pessoa) ? "Pessoa has more than 80 years" : null
        ).filter(Objects::nonNull)
         .toList();

        if (!validationErrors.isEmpty()) {
            throw new ValidationException(validationErrors);
        }
    }

    // Example predicates
    private final Predicate<Pessoa> pessoaExists = pessoa -> /* your logic */;
    private final Predicate<Pessoa> pessoaIsOld = pessoa -> /* your logic */;
}
```

---

### 3. **ControllerAdvice for Exception Handling**

Map the `ValidationException` to a `ProblemDetail` response.

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ProblemDetail;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.util.List;

@RestControllerAdvice
public class ValidationExceptionHandler {

    @ExceptionHandler(ValidationException.class)
    public ProblemDetail handleValidationException(ValidationException ex) {
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(
                HttpStatus.BAD_REQUEST,
                "Validation failed for the request"
        );
        problemDetail.setTitle("Validation Error");

        // Add validation errors to ProblemDetail
        problemDetail.setProperty("errors", ex.getErrors());

        return problemDetail;
    }
}
```

---

### 4. **Controller**

Invoke the service and let the exception flow naturally.

```java
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class PessoaController {

    private final PessoaService pessoaService;

    public PessoaController(PessoaService pessoaService) {
        this.pessoaService = pessoaService;
    }

    @PostMapping("/pessoa")
    public void createPessoa(@RequestBody Pessoa pessoa) {
        pessoaService.validatePessoa(pessoa);
        // Continue with creation logic
    }
}
```

---

### 5. **Best Practices**

1. **Structured Error Responses**: Include both a `title` and `details` for the problem, and attach relevant metadata (e.g., fields, validation messages).
2. **Extendability**: If you want to support field-level validation, enhance the error list to include fields, e.g., `Map<String, String>` where the key is the field name and the value is the error message.
3. **Validation Frameworks**: For complex validations, consider using Hibernate Validator (Bean Validation API) for field-level constraints. However, the `Predicate` approach is still suitable for custom or complex validations.

---

### Example ProblemDetail Response

```json
{
  "type": "about:blank",
  "title": "Validation Error",
  "status": 400,
  "detail": "Validation failed for the request",
  "errors": [
    "Pessoa already exists",
    "Pessoa has more than 80 years"
  ]
}
```

Using the **Bean Validation API** (Hibernate Validator) in Spring Boot is a robust and declarative way to handle validations. Here's how you can implement validations with error mapping into a `ProblemDetail` response:

---

### 1. **Entity with Validation Annotations**

Use annotations like `@NotNull`, `@Size`, or custom constraints to define validation rules.

```java
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Max;

public class Pessoa {

    @NotNull(message = "Name must not be null")
    private String name;

    @Max(value = 80, message = "Age must be less than or equal to 80")
    private Integer age;

    // Getters and Setters
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

---

### 2. **Controller with `@Valid`**

Spring will automatically validate the incoming request body based on the constraints defined in the `Pessoa` class.

```java
import jakarta.validation.Valid;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class PessoaController {

    @PostMapping("/pessoa")
    public String createPessoa(@Valid @RequestBody Pessoa pessoa) {
        // If validation passes, handle business logic
        return "Pessoa created successfully";
    }
}
```

---

### 3. **Global Exception Handler for Validation Errors**

Customize the error response to return a `ProblemDetail` containing validation errors.

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ProblemDetail;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice
public class ValidationExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ProblemDetail handleValidationExceptions(MethodArgumentNotValidException ex) {
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(
                HttpStatus.BAD_REQUEST,
                "Validation failed for the request"
        );
        problemDetail.setTitle("Validation Error");

        // Extract field errors
        Map<String, String> errors = new HashMap<>();
        for (FieldError error : ex.getBindingResult().getFieldErrors()) {
            errors.put(error.getField(), error.getDefaultMessage());
        }

        // Add errors to ProblemDetail
        problemDetail.setProperty("fieldErrors", errors);

        return problemDetail;
    }
}
```

---

### 4. **Example of a POST Request**

#### Input (Invalid)

```json
{
  "name": null,
  "age": 85
}
```

#### Response (Validation Errors)

```json
{
  "type": "about:blank",
  "title": "Validation Error",
  "status": 400,
  "detail": "Validation failed for the request",
  "fieldErrors": {
    "name": "Name must not be null",
    "age": "Age must be less than or equal to 80"
  }
}
```

---

### 5. **Best Practices for Using Validation Frameworks**

1. **Use Declarative Constraints**: Leverage annotations for validation logic to keep it clean and consistent.
2. **Custom Constraints**: For complex rules, create custom annotations with a `ConstraintValidator`.
3. **Error Messages**: Centralize validation messages in a `messages.properties` file for localization.
4. **Controller Advice**: Use a `@ControllerAdvice` to keep validation error handling separate from business logic.
5. **Field-Level Validation**: Return errors mapped to specific fields to improve usability for API consumers.

---

### Optional: Custom Validator Example

If you need a custom validation (e.g., `pessoaExists`), implement a `ConstraintValidator`.

#### Annotation

```java
import jakarta.validation.Constraint;
import jakarta.validation.Payload;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Constraint(validatedBy = PessoaExistsValidator.class)
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidPessoa {
    String message() default "Pessoa already exists";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

#### Validator

```java
import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;

public class PessoaExistsValidator implements ConstraintValidator<ValidPessoa, Pessoa> {

    @Override
    public boolean isValid(Pessoa pessoa, ConstraintValidatorContext context) {
        // Example validation logic
        return !pessoa.getName().equalsIgnoreCase("existing");
    }
}
```

#### Apply Custom Validator

```java
@ValidPessoa
public class Pessoa {
    @NotNull
    private String name;
    private Integer age;

    // Getters and setters...
}
```

This ensures all validation logic is declarative, centralized, and easy to extend.