Handling `404 Not Found` with Feign in a Spring Boot application can be achieved by customizing how Feign processes HTTP status codes. By default, Feign considers `404` and other non-2xx responses as errors and throws exceptions.

Here’s how to handle `404 Not Found` with Feign:

### 1. **Customizing Feign Error Decoder**

You can define a custom `ErrorDecoder` to process the `404` status code differently.

#### Example:

```java
import feign.Response;
import feign.codec.ErrorDecoder;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;

@Component
public class CustomErrorDecoder implements ErrorDecoder {
    
    private final ErrorDecoder defaultErrorDecoder = new Default();

    @Override
    public Exception decode(String methodKey, Response response) {
        if (response.status() == HttpStatus.NOT_FOUND.value()) {
            return new ResourceNotFoundException("Resource not found");
        }
        return defaultErrorDecoder.decode(methodKey, response);
    }
}
```

Here, `ResourceNotFoundException` is a custom exception class:

```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

### 2. **Register the Custom Decoder**

You need to register the custom `ErrorDecoder` in your application.

#### Using a Configuration Class:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FeignConfig {

    @Bean
    public ErrorDecoder errorDecoder() {
        return new CustomErrorDecoder();
    }
}
```

#### Linking Configuration to Feign Client:

```java
@FeignClient(name = "example-client", url = "http://example.com", configuration = FeignConfig.class)
public interface ExampleClient {

    @GetMapping("/resources/{id}")
    Resource getResource(@PathVariable("id") String id);
}
```

### 3. **Handling the Exception**

You can catch the `ResourceNotFoundException` in your service or controller and provide appropriate responses.

#### Example:

```java
@RestController
public class ExampleController {

    private final ExampleClient exampleClient;

    public ExampleController(ExampleClient exampleClient) {
        this.exampleClient = exampleClient;
    }

    @GetMapping("/fetch-resource/{id}")
    public ResponseEntity<?> fetchResource(@PathVariable String id) {
        try {
            Resource resource = exampleClient.getResource(id);
            return ResponseEntity.ok(resource);
        } catch (ResourceNotFoundException ex) {
            return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
        }
    }
}
```

### 4. **Optional: Returning Null for `404`**

If you prefer to return `null` instead of throwing an exception for `404`:

- Use a custom decoder to return `null`:

```java
@Override
public Exception decode(String methodKey, Response response) {
    if (response.status() == HttpStatus.NOT_FOUND.value()) {
        return null;
    }
    return defaultErrorDecoder.decode(methodKey, response);
}
```

- Ensure your Feign method handles `null` responses gracefully.

---

This approach provides a clean way to handle `404 Not Found` while maintaining flexibility and control over Feign error handling in your Spring Boot application.


To add a retry mechanism to Feign requests, you can use **Spring Retry** with Feign. Spring Retry allows you to configure retries for failed requests due to transient issues such as `404 Not Found`, network issues, or other HTTP errors.

Here’s how to add retry functionality:

---

### 1. **Add Spring Retry Dependency**

Add the following dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

---

### 2. **Enable Retry in the Feign Client**

Use the `@Retryable` annotation on the Feign client methods. First, make sure you enable Spring Retry by adding `@EnableRetry` to your Spring Boot application.

#### Update Application Class:

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.retry.annotation.EnableRetry;

@SpringBootApplication
@EnableRetry
public class FeignRetryApplication {
    public static void main(String[] args) {
        SpringApplication.run(FeignRetryApplication.class, args);
    }
}
```

#### Add Retry to Feign Client:

You can configure the retry properties using the `@Retryable` annotation.

```java
import org.springframework.retry.annotation.Retryable;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient(name = "example-client", url = "http://example.com")
public interface ExampleClient {

    @Retryable(
        value = { ResourceNotFoundException.class },
        maxAttempts = 3,
        backoff = @Backoff(delay = 2000)
    )
    @GetMapping("/resources/{id}")
    Resource getResource(@PathVariable("id") String id);
}
```

- `value`: Specifies which exceptions trigger the retry (e.g., `ResourceNotFoundException`).
- `maxAttempts`: The maximum number of retry attempts.
- `backoff`: Adds a delay between retries (e.g., `2000ms`).

---

### 3. **Handle Exhausted Retry Attempts**

To handle cases where all retries fail, you can use `@Recover`. This method is called when the retries are exhausted.

#### Add Recovery Logic:

```java
import org.springframework.retry.annotation.Recover;
import org.springframework.stereotype.Service;

@Service
public class ExampleClientService {

    private final ExampleClient exampleClient;

    public ExampleClientService(ExampleClient exampleClient) {
        this.exampleClient = exampleClient;
    }

    public Resource fetchResource(String id) {
        return exampleClient.getResource(id);
    }

    @Recover
    public Resource recover(ResourceNotFoundException e, String id) {
        System.out.println("All retries exhausted. Returning fallback response.");
        return new Resource("Fallback Resource", "Fallback Description");
    }
}
```

---

### 4. **Fine-Tune Retry Configuration**

Instead of using annotations, you can configure retries globally for Feign clients by using a `Retryer` bean.

#### Global Retry Configuration:

```java
import feign.Retryer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import static java.util.concurrent.TimeUnit.SECONDS;

@Configuration
public class FeignConfig {

    @Bean
    public Retryer retryer() {
        return new Retryer.Default(1000, SECONDS.toMillis(1), 3);
    }
}
```

This will retry the request up to 3 times, with an initial interval of 1 second between retries.

---

### Example Workflow:

1. **Normal Flow**:
    
    - The client retries up to 3 times for transient errors like `404 Not Found`.
    - Backoff ensures a delay between retries.
2. **Fallback Handling**:
    
    - If retries are exhausted, the `@Recover` method provides a fallback response.

---

This configuration ensures robustness by retrying transient errors and gracefully handling exhausted retries with a fallback mechanism.