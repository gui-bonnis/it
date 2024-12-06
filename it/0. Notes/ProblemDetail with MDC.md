Here's an example of using `ProblemDetail` in a Spring Boot application while incorporating **Mapped Diagnostic Context (MDC)** for structured logging. We'll demonstrate handling a custom exception and including MDC data in the response:

### Use Case

You want to:

1. Use `ProblemDetail` for structured error responses.
2. Include MDC values (e.g., `requestId`, `userId`) in logs and error responses for traceability.

---

### 1. Maven Dependencies

Add the following dependencies to your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
</dependency>
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
```

---

### 2. Exception Class

Create a custom exception with relevant details:

```java
public class CustomException extends RuntimeException {
    private final String errorCode;

    public CustomException(String message, String errorCode) {
        super(message);
        this.errorCode = errorCode;
    }

    public String getErrorCode() {
        return errorCode;
    }
}
```

---

### 3. Controller Advice

Handle exceptions and log MDC data.

```java
import org.slf4j.MDC;
import org.springframework.http.HttpStatus;
import org.springframework.http.ProblemDetail;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

import java.time.OffsetDateTime;

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(CustomException.class)
    public ProblemDetail handleCustomException(CustomException ex) {
        // Add structured logging with MDC
        MDC.put("errorCode", ex.getErrorCode());
        MDC.put("timestamp", OffsetDateTime.now().toString());

        // Log the exception
        org.slf4j.LoggerFactory.getLogger(GlobalExceptionHandler.class).error("CustomException occurred: {}", ex.getMessage());

        // Build ProblemDetail response
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, ex.getMessage());
        problemDetail.setTitle("Custom Exception");
        problemDetail.setProperty("errorCode", ex.getErrorCode());
        problemDetail.setProperty("requestId", MDC.get("requestId")); // Include MDC values
        problemDetail.setProperty("timestamp", MDC.get("timestamp"));

        // Clear MDC to prevent leaking context
        MDC.clear();

        return problemDetail;
    }
}
```

---

### 4. Controller

Use MDC to include contextual information for each request.

```java
import org.slf4j.MDC;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;

@RestController
public class SampleController {

    @GetMapping("/test")
    public String testEndpoint(@RequestHeader(value = "User-Id", required = false) String userId) {
        // Populate MDC with contextual data
        MDC.put("requestId", UUID.randomUUID().toString());
        MDC.put("userId", userId != null ? userId : "anonymous");

        // Simulate an error
        throw new CustomException("Something went wrong!", "ERR-001");
    }
}
```

---

### 5. Logback Configuration

Set up `logback-spring.xml` for structured logging:

```xml
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <providers>
                <timestamp />
                <message />
                <loggerName />
                <threadName />
                <mdc />
                <stackTrace />
            </providers>
        </encoder>
    </appender>

    <root level="info">
        <appender-ref ref="CONSOLE" />
    </root>
</configuration>
```

---

### 6. Test the Application

When you call `/test` with a `User-Id` header:

```shell
curl -H "User-Id: 12345" http://localhost:8080/test
```

#### JSON Response:

```json
{
  "type": "about:blank",
  "title": "Custom Exception",
  "status": 400,
  "detail": "Something went wrong!",
  "errorCode": "ERR-001",
  "requestId": "123e4567-e89b-12d3-a456-426614174000",
  "timestamp": "2024-12-02T10:00:00Z"
}
```

#### Logs:

```json
{
  "timestamp": "2024-12-02T10:00:00Z",
  "level": "ERROR",
  "logger": "com.example.GlobalExceptionHandler",
  "message": "CustomException occurred: Something went wrong!",
  "mdc": {
    "errorCode": "ERR-001",
    "timestamp": "2024-12-02T10:00:00Z",
    "requestId": "123e4567-e89b-12d3-a456-426614174000",
    "userId": "12345"
  }
}
```

---

This setup ensures that MDC data is included in both structured logs and API error responses, enhancing traceability across systems.