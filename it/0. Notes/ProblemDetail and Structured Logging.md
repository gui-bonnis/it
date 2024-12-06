### Overview of `ProblemDetail` and Structured Logging

- **`ProblemDetail`**: A class in Java designed to represent HTTP problem details as described in [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807). It provides a standard way to represent error information in API responses.
- **Structured Logging**: Involves logging information in a structured format (e.g., JSON), allowing easier querying, filtering, and analysis using log aggregation tools like ELK Stack or Loki.

---

### Best Practices for `ProblemDetail`

1. **Use Standard Fields**:
    
    - `type`: A URI that identifies the problem type.
    - `title`: A short, human-readable summary of the problem.
    - `status`: The HTTP status code.
    - `detail`: A human-readable explanation of the error.
    - `instance`: A URI reference that identifies the specific occurrence of the problem.
2. **Extend `ProblemDetail`**:
    
    - Include custom fields for application-specific requirements.
    - Use the `withProperty` method (in `ProblemDetail.Builder`) to dynamically add custom properties.
3. **Return Consistent Errors**:
    
    - Standardize the error format across your microservices for uniformity.

---

### Best Practices for Structured Logging

1. **Use a Logging Framework**:
    
    - Use a library like **Logback** or **SLF4J** with appenders configured for JSON output.
2. **Include Contextual Information**:
    
    - Use MDC (Mapped Diagnostic Context) to add contextual metadata (e.g., `requestId`, `userId`).
3. **Log at the Right Level**:
    
    - `DEBUG`: Development-specific details.
    - `INFO`: High-level events (e.g., service start/stop).
    - `WARN`: Recoverable issues.
    - `ERROR`: Critical issues requiring attention.
4. **Avoid Logging Sensitive Data**:
    
    - Mask or redact PII (Personal Identifiable Information).
5. **Use a Correlation ID**:
    
    - Include a unique identifier (e.g., `requestId`) in logs to trace requests across services.

---

### Example Implementation in Spring Boot

#### 1. ProblemDetail Example

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(EntityNotFoundException.class)
    public ResponseEntity<ProblemDetail> handleEntityNotFound(EntityNotFoundException ex, HttpServletRequest request) {
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
        problemDetail.setTitle("Resource Not Found");
        problemDetail.setType(URI.create("https://api.example.com/errors/resource-not-found"));
        problemDetail.setInstance(URI.create(request.getRequestURI()));
        problemDetail.withProperty("errorCode", "ENTITY_NOT_FOUND");
        problemDetail.withProperty("timestamp", Instant.now());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(problemDetail);
    }
}
```

#### Sample Response

```json
{
  "type": "https://api.example.com/errors/resource-not-found",
  "title": "Resource Not Found",
  "status": 404,
  "detail": "The requested entity was not found.",
  "instance": "/api/orders/123",
  "errorCode": "ENTITY_NOT_FOUND",
  "timestamp": "2024-12-03T10:15:30Z"
}
```

---

#### 2. Structured Logging Example

**Logback Configuration (`logback-spring.xml`)**:

```xml
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <providers>
                <timestamp>
                    <timeZone>UTC</timeZone>
                </timestamp>
                <pattern>
                    <pattern>message</pattern>
                </pattern>
                <mdc />
                <context />
                <stackTrace />
            </providers>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE" />
    </root>
</configuration>
```

**Adding Context Using MDC**:

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    private final Logger logger = LoggerFactory.getLogger(OrderController.class);

    @GetMapping("/{orderId}")
    public ResponseEntity<Order> getOrder(@PathVariable String orderId) {
        MDC.put("requestId", UUID.randomUUID().toString());
        MDC.put("userId", "user-123"); // Example, replace with actual user data
        logger.info("Fetching order with ID {}", orderId);

        // Simulate order fetching logic
        Order order = new Order(orderId, "Sample Item", 2);
        logger.debug("Order details: {}", order);

        MDC.clear();
        return ResponseEntity.ok(order);
    }
}
```

**Sample Log Output**:

```json
{
  "timestamp": "2024-12-03T10:15:30.123Z",
  "message": "Fetching order with ID 123",
  "logger_name": "com.example.demo.OrderController",
  "level": "INFO",
  "thread_name": "http-nio-8080-exec-1",
  "requestId": "9b8e3f1a-f67d-4d88-861d-a015f5cc123a",
  "userId": "user-123"
}
```

---

### State-of-the-Art Practices

1. **Integrate Distributed Tracing**:
    
    - Use tools like **Jaeger** or **Zipkin** for tracing distributed requests across microservices.
2. **Leverage Centralized Log Management**:
    
    - Use **ELK (Elasticsearch, Logstash, Kibana)**, **Loki**, or **Splunk** for centralized log aggregation and querying.
3. **Adopt Observability**:
    
    - Combine structured logging with metrics (e.g., Prometheus) and tracing for complete observability.
4. **Use Fluentd or Logstash**:
    
    - Forward logs in real time to your log management solution.

This combination of `ProblemDetail` and structured logging ensures a robust and maintainable approach to error handling and observability in modern applications.


For complex business applications, structured logging plays a critical role in observability and debugging. Integrating with tools like Fluentd, Jaeger, and Prometheus requires a strategic approach to log structure, distributed tracing, and application-level metrics. Here's how you can achieve this:

---

### **1. Designing Structured Logs for Fluentd**

Fluentd requires logs to be in a structured format (JSON) for optimal parsing and forwarding.

#### Example Log Structure

```json
{
  "timestamp": "2024-12-03T10:15:30.123Z",
  "level": "INFO",
  "message": "Order processed successfully",
  "service": "order-service",
  "spanId": "b7431c63d9e57b5d",
  "traceId": "4a7dbdf1c9e23c7e",
  "businessContext": {
    "orderId": "12345",
    "userId": "user-789",
    "totalAmount": 150.75
  }
}
```

### Fluentd Configuration

Fluentd can forward logs to Elasticsearch, Loki, or cloud logging solutions. Below is a basic configuration to capture logs:

```yaml
<source>
  @type tail
  path /var/log/app/*.log
  format json
  tag application.logs
</source>

<match application.logs>
  @type elasticsearch
  host elasticsearch.local
  port 9200
  logstash_format true
</match>
```

### Code Implementation in Spring Boot

Configure logs to emit JSON and include MDC data for business context:

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    private static final Logger logger = LoggerFactory.getLogger(OrderController.class);

    @PostMapping
    public ResponseEntity<String> processOrder(@RequestBody OrderRequest orderRequest) {
        MDC.put("traceId", "generated-trace-id"); // Replace with trace ID from Jaeger
        MDC.put("spanId", "generated-span-id"); // Replace with span ID
        MDC.put("userId", orderRequest.getUserId());

        logger.info("Processing order with ID {}", orderRequest.getOrderId());

        // Process order logic
        logger.info("Order processed successfully for user {}", orderRequest.getUserId());

        MDC.clear();
        return ResponseEntity.ok("Order processed");
    }
}
```

Configure Logback for JSON output:

```xml
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <providers>
                <timestamp />
                <pattern>
                    <pattern>message</pattern>
                </pattern>
                <mdc />
                <context />
            </providers>
        </encoder>
    </appender>
    <root level="INFO">
        <appender-ref ref="CONSOLE" />
    </root>
</configuration>
```

---

### **2. Distributed Tracing with Jaeger**

Jaeger tracks the flow of requests across microservices. Use OpenTelemetry to instrument your application.

#### OpenTelemetry Setup

1. **Add Dependencies**: Add the OpenTelemetry SDK and Jaeger exporter to your `pom.xml`:
    
    ```xml
    <dependency>
        <groupId>io.opentelemetry</groupId>
        <artifactId>opentelemetry-exporter-jaeger</artifactId>
        <version>1.29.0</version>
    </dependency>
    ```
    
2. **Initialize Tracing**: Configure the tracer in your application:
    
    ```java
    import io.opentelemetry.api.GlobalOpenTelemetry;
    import io.opentelemetry.api.trace.Tracer;
    import io.opentelemetry.api.trace.Span;
    import io.opentelemetry.context.Scope;
    
    public class TracingExample {
        private static final Tracer tracer = GlobalOpenTelemetry.getTracer("order-service");
    
        public void processOrder(String orderId) {
            Span span = tracer.spanBuilder("processOrder").startSpan();
            try (Scope scope = span.makeCurrent()) {
                span.setAttribute("order.id", orderId);
                span.setAttribute("user.id", "user-123");
                // Process order logic
            } finally {
                span.end();
            }
        }
    }
    ```
    
3. **Jaeger Configuration**: Export traces to Jaeger by setting the following environment variables:
    
    ```bash
    OTEL_EXPORTER_JAEGER_ENDPOINT=http://jaeger-collector:14250
    OTEL_SERVICE_NAME=order-service
    ```
    

---

### **3. Application-Level Metrics with Prometheus**

Prometheus collects and queries metrics. Use the **Micrometer** library for metrics instrumentation.

#### Add Dependency

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
    <version>1.11.0</version>
</dependency>
```

#### Example Metrics

Instrument your code to collect business-specific metrics:

```java
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.stereotype.Service;

@Service
public class OrderService {

    private final MeterRegistry meterRegistry;

    public OrderService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }

    public void processOrder(OrderRequest orderRequest) {
        meterRegistry.counter("orders.processed", "userId", orderRequest.getUserId())
                     .increment();
        // Order processing logic
        meterRegistry.timer("order.processing.time").record(() -> {
            // Simulate processing
        });
    }
}
```

#### Prometheus Configuration

Spring Boot exposes metrics at `/actuator/prometheus`. Configure Prometheus to scrape metrics:

```yaml
scrape_configs:
  - job_name: 'order-service'
    static_configs:
      - targets: ['localhost:8080']
```

---

### **4. Observability Flow**

- **Structured Logs (Fluentd)**: Capture application logs enriched with trace IDs and business context.
- **Distributed Tracing (Jaeger)**: Trace requests across services using OpenTelemetry.
- **Application Metrics (Prometheus)**: Collect business metrics like order processing time, error rates, etc.

---

### **5. Summary**

|Tool|Purpose|Code/Configuration|
|---|---|---|
|**Fluentd**|Centralized logging|Logback for JSON logs|
|**Jaeger**|Distributed tracing|OpenTelemetry instrumentation|
|**Prometheus**|Application metrics|Micrometer for business-specific metrics|

This setup ensures a robust observability stack with clear insights into application behavior and performance.


Here’s a detailed dive into **what information** to collect, **why it’s important**, and **when to log or emit it** for structured logs, distributed tracing, and application-level metrics. Each section considers the **phases of a request lifecycle**, from entry into your system to its completion.

---

### **1. Structured Logs**

Structured logs capture a **point-in-time snapshot** of your application state. They should provide contextual details to aid debugging and operational insights.

#### **What to Log**

- **Core Context** (present in all logs):
    
    - `timestamp`: The time the event occurred (ISO 8601 format).
    - `level`: Log severity (`INFO`, `DEBUG`, `WARN`, `ERROR`).
    - `message`: The main log message.
    - `service`: The service or application name.
    - `environment`: Deployment environment (e.g., `dev`, `staging`, `prod`).
- **Request-Level Information**:
    
    - `traceId`, `spanId`: IDs for distributed tracing correlation.
    - `requestId`: A unique ID for the current request.
    - `endpoint`: The REST endpoint or resource accessed.
    - `method`: HTTP method (GET, POST, etc.).
    - `statusCode`: HTTP response status code.
    - `duration`: Time taken to process the request.
- **Business Context**:
    
    - `userId`, `customerId`: IDs for the current user or customer.
    - `orderId`, `transactionId`: Any relevant business identifiers.
- **Error Details**:
    
    - `exceptionType`: Type of the exception thrown.
    - `exceptionMessage`: Exception message.
    - `stackTrace`: Stack trace for debugging (log only at `ERROR` level).

---

#### **Why to Log It**

- **Core Context**: Helps filter and query logs by service, environment, and time.
- **Request-Level Information**: Enables tracing the lifecycle of a request and identifying bottlenecks or failures.
- **Business Context**: Allows tracking specific business actions, which is crucial for domain-level debugging.
- **Error Details**: Provides actionable information for debugging exceptions.

---

#### **When to Log**

- **INFO Level**:
    - At service entry and exit (e.g., before/after controller methods).
    - For significant business events (e.g., order creation, payment processed).
- **DEBUG Level**:
    - For detailed insights into application logic (e.g., loop iterations, configuration values).
- **WARN Level**:
    - For recoverable issues (e.g., fallback logic, degraded service response).
- **ERROR Level**:
    - For exceptions or failures (e.g., database connection issues, API timeouts).

---

### **2. Distributed Tracing**

Distributed tracing tracks the flow of requests across services, focusing on latency and errors.

#### **What to Collect**

- **Core Trace Information**:
    
    - `traceId`: Unique ID for the entire request lifecycle.
    - `spanId`: ID for the current operation in the trace.
    - `parentSpanId`: ID of the calling span (if any).
    - `timestamp`: Start and end time of the span.
- **Attributes**:
    
    - **Request Details**:
        - `http.method`: HTTP method used (GET, POST, etc.).
        - `http.url`: Full URL accessed.
        - `http.status_code`: HTTP status code.
        - `http.client_ip`: Client IP address.
    - **Application Details**:
        - `service.name`: Service name emitting the trace.
        - `service.version`: Service version.
        - `region`, `environment`: Deployment metadata.
    - **Business Details**:
        - Custom attributes like `userId`, `orderId`, or `transactionAmount`.
- **Events**:
    
    - Key lifecycle events like retries, errors, or database queries.

---

#### **Why to Collect It**

- **Trace Information**: Correlates all service calls in a request, enabling latency analysis.
- **Attributes**: Enriches trace context, making it easier to diagnose specific issues (e.g., identifying slow services).
- **Events**: Highlights important milestones within a span (e.g., retries, exception handling).

---

#### **When to Emit**

- **Span Creation**:
    - At every service boundary (e.g., controller entry, outgoing HTTP calls).
- **Span Completion**:
    - When the operation finishes (successful or failed).
- **Custom Events**:
    - When significant actions occur (e.g., retries, external API calls).

---

### **3. Application-Level Metrics**

Metrics provide **quantitative insights** into your application’s performance and behavior.

#### **What to Measure**

- **Request Metrics**:
    
    - **Counts**:
        - `http_requests_total`: Total number of HTTP requests received.
        - `http_errors_total`: Total number of failed requests.
    - **Latencies**:
        - `http_request_duration_seconds`: Distribution of request durations.
- **Business Metrics**:
    
    - `orders_placed_total`: Total number of orders placed.
    - `orders_failed_total`: Total number of failed orders.
    - `revenue_total`: Total revenue processed.
- **System Metrics**:
    
    - `cpu_usage`: CPU usage percentage.
    - `memory_usage`: Memory usage in MB.
    - `disk_io`: Disk read/write rates.
- **Custom Metrics**:
    
    - Domain-specific insights (e.g., inventory levels, retry counts).

---

#### **Why to Measure It**

- **Request Metrics**:
    - Track service availability and performance (e.g., request latencies, error rates).
- **Business Metrics**:
    - Monitor domain health (e.g., order volume trends, revenue anomalies).
- **System Metrics**:
    - Identify infrastructure bottlenecks (e.g., CPU spikes, memory leaks).

---

#### **When to Emit**

- **Request Metrics**:
    - At the start and end of each HTTP request.
- **Business Metrics**:
    - After completing a business action (e.g., order processed, payment authorized).
- **System Metrics**:
    - At regular intervals (e.g., every 10 seconds).

---

### **Summary Table**

|**Category**|**What**|**Why**|**When**|
|---|---|---|---|
|**Structured Logs**|Core context, request/business info, errors|Debugging, operational insights|Service entry/exit, business events|
|**Distributed Tracing**|Trace IDs, spans, attributes, events|Latency analysis, dependency mapping|Service boundaries, significant events|
|**Metrics**|Request, business, and system metrics|Quantitative performance and domain insights|Request completion, periodic emission|

---

### **Example Integration: Unified Observability**

1. **Structured Logs**:
    
    - Use structured JSON logs (Logback or SLF4J) to emit logs with trace IDs and business context.
2. **Distributed Tracing**:
    
    - Use OpenTelemetry to instrument traces and propagate trace headers (`traceparent`, `tracestate`) across services.
3. **Application Metrics**:
    
    - Use Micrometer to emit metrics for Prometheus.

By ensuring logs, traces, and metrics align and complement each other, you build a cohesive observability system that provides both **granular details** and **high-level insights**.


Here’s an example of a **structured log** for a complex business application using JSON format. This log captures the **core context**, **request-level information**, **business context**, and **error details** when applicable.

---

### **Example: Successful Request**

```json
{
  "timestamp": "2024-12-03T14:15:22.345Z",
  "level": "INFO",
  "message": "Order successfully created",
  "service": "order-service",
  "environment": "production",
  "request": {
    "traceId": "a123b456c789",
    "spanId": "span123",
    "parentSpanId": "parentSpan123",
    "requestId": "req-45678",
    "endpoint": "/api/v1/orders",
    "method": "POST",
    "statusCode": 201,
    "duration": 342
  },
  "businessContext": {
    "userId": "user-12345",
    "customerId": "cust-67890",
    "orderId": "ord-98765",
    "transactionAmount": 120.75,
    "currency": "USD"
  }
}
```

---

### **Example: Failed Request (with Error Details)**

```json
{
  "timestamp": "2024-12-03T14:16:42.567Z",
  "level": "ERROR",
  "message": "Failed to create order",
  "service": "order-service",
  "environment": "production",
  "request": {
    "traceId": "a123b456c789",
    "spanId": "span789",
    "parentSpanId": "parentSpan123",
    "requestId": "req-56789",
    "endpoint": "/api/v1/orders",
    "method": "POST",
    "statusCode": 500,
    "duration": 500,
    "clientIp": "192.168.1.50",
    "userAgent": "PostmanRuntime/7.29.0"
  },
  "businessContext": {
    "userId": "user-12345",
    "customerId": "cust-67890",
    "transactionAmount": 120.75,
    "currency": "USD"
  },
  "error": {
    "exceptionType": "OrderCreationException",
    "exceptionMessage": "Database timeout while saving order",
    "stackTrace": "com.example.orders.OrderService.createOrder(OrderService.java:123)"
  }
}
```

---

### **Breaking Down the Log**

#### **Core Context**

- **`timestamp`**: Ensures precise ordering of events.
- **`level`**: Indicates the severity (`INFO`, `ERROR`).
- **`message`**: Descriptive information about the log event.
- **`service` & `environment`**: Identifies the application and its deployment environment.

#### **Request-Level Information**

- **Traceability**:
    - `traceId`, `spanId`, `parentSpanId`: Links logs with distributed traces.
    - `requestId`: Uniquely identifies the request.
- **Endpoint and Method**: Specifies the accessed REST endpoint and HTTP method.
- **Response Details**:
    - `statusCode`: HTTP status returned by the application.
    - `duration`: Time taken to handle the request (in ms).
- **Client Details** (optional):
    - `clientIp`, `userAgent`: Useful for debugging client-specific issues.

#### **Business Context**

- Captures domain-specific details such as `userId`, `customerId`, `orderId`, and `transactionAmount`, which are vital for understanding the event in the business workflow.

#### **Error Details**

- **`exceptionType`**: Type of exception thrown.
- **`exceptionMessage`**: Clear explanation of the error.
- **`stackTrace`**: Optional for `ERROR` level to assist in debugging.

---

### **How It Fits into Observability**

- **Correlation**:
    - Use `traceId` and `spanId` to link structured logs with distributed traces in systems like Jaeger or Zipkin.
- **Searchability**:
    - JSON logs are easily queryable using tools like Elasticsearch, FluentD, or Splunk.
- **Domain Insights**:
    - Capturing business context (e.g., `transactionAmount`) allows combining operational monitoring with domain-specific KPIs.

By adopting such structured logging practices, logs become a powerful tool for debugging, analytics, and proactive monitoring.

Here’s an example of **Java code** for logging the success and error scenarios in a structured format using the popular **SLF4J with Logback** library. It uses **MDC (Mapped Diagnostic Context)** to handle properties like `traceId` and `spanId` consistently.

---

### **Dependencies**

Add the following dependencies to your `pom.xml`:

```xml
<dependencies>
    <!-- SLF4J API -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>2.0.9</version>
    </dependency>
    <!-- Logback for Structured Logging -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.4.8</version>
    </dependency>
    <!-- JSON Logging Support -->
    <dependency>
        <groupId>net.logstash.logback</groupId>
        <artifactId>logstash-logback-encoder</artifactId>
        <version>7.3</version>
    </dependency>
</dependencies>
```

---

### **Logback Configuration (`logback-spring.xml`)**

Use Logstash encoder for structured JSON logs.

```xml
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <!-- Include MDC properties in structured logs -->
            <includeMdcKeyName>request</includeMdcKeyName>
            <includeMdcKeyName>businessContext</includeMdcKeyName>
            <includeMdcKeyName>error</includeMdcKeyName>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE" />
    </root>
</configuration>
```

---

### **Java Code for Structured Logging**

#### **Success Scenario**

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;

import java.util.UUID;

public class StructuredLoggingExample {
    private static final Logger logger = LoggerFactory.getLogger(StructuredLoggingExample.class);

    public static void logSuccess() {
        try {
            // Core Context
            MDC.put("timestamp", "2024-12-03T14:15:22.345Z");
            MDC.put("level", "INFO");
            MDC.put("service", "order-service");
            MDC.put("environment", "production");

            // Request Information
            MDC.put("request.traceId", UUID.randomUUID().toString());
            MDC.put("request.spanId", UUID.randomUUID().toString());
            MDC.put("request.parentSpanId", "parentSpan123");
            MDC.put("request.requestId", "req-45678");
            MDC.put("request.endpoint", "/api/v1/orders");
            MDC.put("request.method", "POST");
            MDC.put("request.statusCode", "201");
            MDC.put("request.duration", "342");

            // Business Context
            MDC.put("businessContext.userId", "user-12345");
            MDC.put("businessContext.customerId", "cust-67890");
            MDC.put("businessContext.orderId", "ord-98765");
            MDC.put("businessContext.transactionAmount", "120.75");
            MDC.put("businessContext.currency", "USD");

            logger.info("Order successfully created");
        } finally {
            MDC.clear();
        }
    }
}
```

---

#### **Error Scenario**

```java
public class StructuredLoggingExample {
    public static void logError() {
        try {
            // Core Context
            MDC.put("timestamp", "2024-12-03T14:16:42.567Z");
            MDC.put("level", "ERROR");
            MDC.put("service", "order-service");
            MDC.put("environment", "production");

            // Request Information
            MDC.put("request.traceId", UUID.randomUUID().toString());
            MDC.put("request.spanId", UUID.randomUUID().toString());
            MDC.put("request.parentSpanId", "parentSpan123");
            MDC.put("request.requestId", "req-56789");
            MDC.put("request.endpoint", "/api/v1/orders");
            MDC.put("request.method", "POST");
            MDC.put("request.statusCode", "500");
            MDC.put("request.duration", "500");
            MDC.put("request.clientIp", "192.168.1.50");
            MDC.put("request.userAgent", "PostmanRuntime/7.29.0");

            // Business Context
            MDC.put("businessContext.userId", "user-12345");
            MDC.put("businessContext.customerId", "cust-67890");
            MDC.put("businessContext.transactionAmount", "120.75");
            MDC.put("businessContext.currency", "USD");

            // Error Details
            MDC.put("error.exceptionType", "OrderCreationException");
            MDC.put("error.exceptionMessage", "Database timeout while saving order");
            MDC.put("error.stackTrace", "com.example.orders.OrderService.createOrder(OrderService.java:123)");

            logger.error("Failed to create order");
        } finally {
            MDC.clear();
        }
    }
}
```

---

### **Usage Example**

```java
public class Main {
    public static void main(String[] args) {
        StructuredLoggingExample.logSuccess();
        StructuredLoggingExample.logError();
    }
}
```

---

### **Expected Output**

This will generate structured JSON logs like:

#### **Success Log**

```json
{
  "timestamp": "2024-12-03T14:15:22.345Z",
  "level": "INFO",
  "message": "Order successfully created",
  "service": "order-service",
  "environment": "production",
  "request": {
    "traceId": "a123b456c789",
    "spanId": "span123",
    "parentSpanId": "parentSpan123",
    "requestId": "req-45678",
    "endpoint": "/api/v1/orders",
    "method": "POST",
    "statusCode": 201,
    "duration": 342
  },
  "businessContext": {
    "userId": "user-12345",
    "customerId": "cust-67890",
    "orderId": "ord-98765",
    "transactionAmount": 120.75,
    "currency": "USD"
  }
}
```

#### **Error Log**

```json
{
  "timestamp": "2024-12-03T14:16:42.567Z",
  "level": "ERROR",
  "message": "Failed to create order",
  "service": "order-service",
  "environment": "production",
  "request": {
    "traceId": "a123b456c789",
    "spanId": "span789",
    "parentSpanId": "parentSpan123",
    "requestId": "req-56789",
    "endpoint": "/api/v1/orders",
    "method": "POST",
    "statusCode": 500,
    "duration": 500,
    "clientIp": "192.168.1.50",
    "userAgent": "PostmanRuntime/7.29.0"
  },
  "businessContext": {
    "userId": "user-12345",
    "customerId": "cust-67890",
    "transactionAmount": 120.75,
    "currency": "USD"
  },
  "error": {
    "exceptionType": "OrderCreationException",
    "exceptionMessage": "Database timeout while saving order",
    "stackTrace": "com.example.orders.OrderService.createOrder(OrderService.java:123)"
  }
}
```

This approach ensures logs are machine-readable and include all relevant context for observability and debugging.

### **Distributed Tracing in Action**

Here’s an example implementation of **Distributed Tracing** using **OpenTelemetry** (Java) integrated with **Jaeger**. This example demonstrates how to trace a request across two services: an `OrderService` and an `InventoryService`.

---

### **Dependencies**

Add the following dependencies to the `pom.xml`:

```xml
<dependencies>
    <!-- OpenTelemetry API -->
    <dependency>
        <groupId>io.opentelemetry</groupId>
        <artifactId>opentelemetry-api</artifactId>
        <version>1.27.0</version>
    </dependency>
    <!-- OpenTelemetry SDK -->
    <dependency>
        <groupId>io.opentelemetry</groupId>
        <artifactId>opentelemetry-sdk</artifactId>
        <version>1.27.0</version>
    </dependency>
    <!-- OpenTelemetry Jaeger Exporter -->
    <dependency>
        <groupId>io.opentelemetry</groupId>
        <artifactId>opentelemetry-exporter-jaeger</artifactId>
        <version>1.27.0</version>
    </dependency>
    <!-- OpenTelemetry Servlet Instrumentation -->
    <dependency>
        <groupId>io.opentelemetry.instrumentation</groupId>
        <artifactId>opentelemetry-servlet-3.0</artifactId>
        <version>1.27.0</version>
    </dependency>
    <!-- OpenTelemetry HTTP Client Instrumentation -->
    <dependency>
        <groupId>io.opentelemetry.instrumentation</groupId>
        <artifactId>opentelemetry-httpclient</artifactId>
        <version>1.27.0</version>
    </dependency>
</dependencies>
```

---

### **Service 1: OrderService**

This service handles a request to create an order and communicates with the `InventoryService` to check stock availability.

#### **Configuration: OpenTelemetry Setup**

```java
import io.opentelemetry.api.GlobalOpenTelemetry;
import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.exporter.jaeger.JaegerGrpcSpanExporter;
import io.opentelemetry.sdk.OpenTelemetrySdk;
import io.opentelemetry.sdk.trace.SdkTracerProvider;
import io.opentelemetry.sdk.trace.export.BatchSpanProcessor;

public class OpenTelemetryConfig {
    public static Tracer initTracer() {
        // Configure Jaeger Exporter
        JaegerGrpcSpanExporter jaegerExporter = JaegerGrpcSpanExporter.builder()
                .setEndpoint("http://localhost:14250") // Jaeger Collector endpoint
                .build();

        // Configure SDK Tracer
        SdkTracerProvider tracerProvider = SdkTracerProvider.builder()
                .addSpanProcessor(BatchSpanProcessor.builder(jaegerExporter).build())
                .build();

        OpenTelemetrySdk openTelemetry = OpenTelemetrySdk.builder()
                .setTracerProvider(tracerProvider)
                .build();

        GlobalOpenTelemetry.set(openTelemetry);

        return openTelemetry.getTracer("order-service-tracer");
    }
}
```

---

#### **Controller: Handling Order Creation**

```java
import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.Tracer;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.net.URI;

@RestController
public class OrderController {
    private final Tracer tracer;

    public OrderController() {
        this.tracer = OpenTelemetryConfig.initTracer();
    }

    @GetMapping("/create-order")
    public String createOrder() throws Exception {
        Span span = tracer.spanBuilder("create-order-span").startSpan();
        try {
            // Simulate checking inventory
            String inventoryResponse = checkInventory();

            if ("IN_STOCK".equals(inventoryResponse)) {
                span.setAttribute("order.status", "SUCCESS");
                return "Order created successfully";
            } else {
                span.setAttribute("order.status", "FAILED");
                return "Order failed: Out of stock";
            }
        } finally {
            span.end();
        }
    }

    private String checkInventory() throws Exception {
        HttpClient httpClient = HttpClient.newHttpClient();
        HttpRequest request = HttpRequest.newBuilder()
                .uri(new URI("http://localhost:8081/check-inventory"))
                .GET()
                .build();

        HttpResponse<String> response = httpClient.send(request, HttpResponse.BodyHandlers.ofString());
        return response.body();
    }
}
```

---

### **Service 2: InventoryService**

This service checks inventory status for the requested item.

#### **Controller: Checking Inventory**

```java
import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.Tracer;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class InventoryController {
    private final Tracer tracer;

    public InventoryController() {
        this.tracer = OpenTelemetryConfig.initTracer();
    }

    @GetMapping("/check-inventory")
    public String checkInventory() {
        Span span = tracer.spanBuilder("check-inventory-span").startSpan();
        try {
            // Simulate inventory check
            span.setAttribute("inventory.itemId", "item-12345");
            span.setAttribute("inventory.quantity", 10);

            return "IN_STOCK";
        } finally {
            span.end();
        }
    }
}
```

---

### **Setting Up Jaeger**

1. Run Jaeger via Docker:
    
    ```bash
    docker run -d --name jaeger \
      -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 \
      -p 5775:5775/udp \
      -p 6831:6831/udp \
      -p 6832:6832/udp \
      -p 5778:5778 \
      -p 16686:16686 \
      -p 14268:14268 \
      -p 14250:14250 \
      -p 9411:9411 \
      jaegertracing/all-in-one:1.46
    ```
    
2. Access Jaeger UI: [http://localhost:16686](http://localhost:16686/).
    

---

### **How It Works**

1. **OrderService** creates a span for processing the `/create-order` request.
2. It invokes **InventoryService** using an HTTP call, which creates its own span for the `/check-inventory` request.
3. The spans are linked using the `traceId` to form a distributed trace.

---

### **Jaeger Visualization**

You will see a trace in Jaeger that looks like this:

- **TraceId**: Shared across both services.
- **Spans**:
    - `create-order-span` from OrderService.
    - `check-inventory-span` from InventoryService.

Each span includes attributes (`inventory.itemId`, `order.status`) and timestamps, giving full visibility into the request flow.

---

This implementation ensures distributed tracing across microservices, enabling visibility into the entire lifecycle of a request and helping to diagnose performance bottlenecks or failures.

Integrating distributed tracing with **exception handling** and **Aspect-Oriented Programming (AOP)** can significantly enhance observability by ensuring all errors and key cross-cutting concerns are captured within your traces. Here's an explanation of why and how to implement these integrations.

---

### **Why Integrate Distributed Tracing with Exception Handling and AOP?**

#### **Exception Handling Integration**

- To ensure all exceptions (handled or unhandled) are recorded as part of the trace for easier debugging.
- To capture details such as exception type, message, and stack trace in the active span.

#### **AOP Integration**

- To reduce boilerplate by automatically adding tracing for cross-cutting concerns such as logging, performance monitoring, or security.
- To ensure every method (especially service layer methods) is traced consistently without manually annotating every method.

---

### **Implementation Steps**

#### **1. Integrate Tracing with Exception Handling**

Use a global exception handler to capture and record exceptions into the active span.

##### **Implementation**

```java
import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.StatusCode;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@ControllerAdvice
public class GlobalExceptionHandler {
    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleException(Exception exception) {
        // Log the exception
        logger.error("Exception occurred: {}", exception.getMessage(), exception);

        // Capture exception in the active trace span
        Span currentSpan = Span.current();
        if (currentSpan != null) {
            currentSpan.recordException(exception);
            currentSpan.setStatus(StatusCode.ERROR, "Exception: " + exception.getMessage());
            currentSpan.setAttribute("exception.type", exception.getClass().getName());
            currentSpan.setAttribute("exception.message", exception.getMessage());
        }

        return new ResponseEntity<>("An error occurred: " + exception.getMessage(), HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

---

#### **2. Integrate Tracing with AOP**

Use AOP to automatically start spans for selected methods or layers.

##### **Implementation**

```java
import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.Tracer;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class TracingAspect {
    private final Tracer tracer;

    public TracingAspect(Tracer tracer) {
        this.tracer = tracer;
    }

    @Around("@annotation(Traceable) || execution(* com.example.service..*(..))")
    public Object traceMethod(ProceedingJoinPoint joinPoint) throws Throwable {
        // Get method name and arguments
        String methodName = joinPoint.getSignature().toShortString();

        // Start a new span
        Span span = tracer.spanBuilder(methodName).startSpan();

        try {
            // Add attributes for arguments
            Object[] args = joinPoint.getArgs();
            for (int i = 0; i < args.length; i++) {
                span.setAttribute("arg" + i, String.valueOf(args[i]));
            }

            // Proceed with the method execution
            return joinPoint.proceed();
        } catch (Throwable throwable) {
            // Record exception in the span
            span.recordException(throwable);
            span.setStatus(StatusCode.ERROR, "Exception in " + methodName);

            throw throwable; // Rethrow exception for further handling
        } finally {
            // End the span
            span.end();
        }
    }
}
```

##### **Custom Annotation: `@Traceable`**

To use AOP selectively, you can define a custom annotation:

```java
package com.example.annotations;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface Traceable {
}
```

You can then annotate methods or classes like so:

```java
@Traceable
public void someBusinessLogic() {
    // Business logic
}
```

---

### **Combined Workflow**

1. **Exception Handling**:
    - Any unhandled exception is captured in the global exception handler and added to the active span.
2. **AOP**:
    - Any method annotated with `@Traceable` or matching a defined pointcut (e.g., service layer methods) is automatically traced, including arguments, execution time, and exceptions.

---

### **Best Practices**

- **Granularity**: Avoid tracing very short-lived or non-critical methods to minimize overhead.
- **Exception Details**: Limit recorded stack traces to avoid overloading your tracing backend (e.g., Jaeger).
- **Span Naming**: Use descriptive names for spans, such as `className.methodName`, for better searchability in Jaeger.
- **Selective Instrumentation**: Use annotations like `@Traceable` to trace only key business methods.

---

### **End-to-End Visibility Example**

- **Controller Layer**: A `Span` is created for the REST API endpoint.
- **Service Layer (via AOP)**: Another `Span` wraps business logic methods automatically.
- **Exception Handling**: Any exception during the request flow is recorded in the active `Span`.

By combining exception handling and AOP with distributed tracing, you ensure consistent and comprehensive tracing throughout your application.


Here's a detailed example of implementing **Application-Level Metrics** in a Spring Boot application using **Micrometer** as the instrumentation library and **Prometheus** as the metrics backend.

---

### **Implementation Overview**

We'll create custom metrics for:

1. **Tracking Order Creation** (success count, failure count, average processing time).
2. **Exception Counts** (categorized by exception type).
3. **Active Users** (a gauge to measure the number of users actively interacting with the system).

---

### **Dependencies**

Add the required dependencies for Micrometer and Prometheus in your `pom.xml`:

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

---

### **Code Implementation**

#### **1. Metrics Configuration**

Configure the Prometheus metrics endpoint in `application.properties`:

```properties
management.endpoints.web.exposure.include=prometheus
management.metrics.export.prometheus.enabled=true
management.endpoint.prometheus.enabled=true
```

---

#### **2. Metric Registration**

Create a bean to define and manage custom metrics.

```java
import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.DistributionSummary;
import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Gauge;
import org.springframework.stereotype.Component;

import java.util.concurrent.atomic.AtomicInteger;

@Component
public class MetricsRegistry {
    private final Counter orderSuccessCounter;
    private final Counter orderFailureCounter;
    private final DistributionSummary orderProcessingTime;
    private final AtomicInteger activeUsersGauge;

    public MetricsRegistry(MeterRegistry meterRegistry) {
        // Counters for order success and failure
        this.orderSuccessCounter = Counter.builder("orders.success")
                .description("Number of successfully processed orders")
                .register(meterRegistry);

        this.orderFailureCounter = Counter.builder("orders.failure")
                .description("Number of failed order processing attempts")
                .register(meterRegistry);

        // Distribution summary for processing time
        this.orderProcessingTime = DistributionSummary.builder("orders.processing.time")
                .description("Order processing time in milliseconds")
                .baseUnit("milliseconds")
                .register(meterRegistry);

        // Gauge for active users
        this.activeUsersGauge = meterRegistry.gauge("users.active", new AtomicInteger(0));
    }

    public void incrementOrderSuccess() {
        orderSuccessCounter.increment();
    }

    public void incrementOrderFailure() {
        orderFailureCounter.increment();
    }

    public void recordProcessingTime(double timeMillis) {
        orderProcessingTime.record(timeMillis);
    }

    public AtomicInteger getActiveUsersGauge() {
        return activeUsersGauge;
    }
}
```

---

#### **3. Instrumenting Business Logic**

Add metrics tracking to your business logic.

```java
import org.springframework.stereotype.Service;

@Service
public class OrderService {
    private final MetricsRegistry metricsRegistry;

    public OrderService(MetricsRegistry metricsRegistry) {
        this.metricsRegistry = metricsRegistry;
    }

    public void processOrder(String orderId) {
        long startTime = System.currentTimeMillis();

        try {
            // Simulate order processing logic
            Thread.sleep((long) (Math.random() * 500));

            // Record success
            metricsRegistry.incrementOrderSuccess();
        } catch (Exception e) {
            // Record failure
            metricsRegistry.incrementOrderFailure();
            throw new RuntimeException("Order processing failed", e);
        } finally {
            // Record processing time
            long duration = System.currentTimeMillis() - startTime;
            metricsRegistry.recordProcessingTime(duration);
        }
    }
}
```

---

#### **4. Active Users Gauge**

Use the active users gauge to track the number of users currently interacting with the application.

```java
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/users")
public class UserController {
    private final MetricsRegistry metricsRegistry;

    public UserController(MetricsRegistry metricsRegistry) {
        this.metricsRegistry = metricsRegistry;
    }

    @PostMapping("/login")
    public String login(@RequestParam String userId) {
        metricsRegistry.getActiveUsersGauge().incrementAndGet();
        return "User " + userId + " logged in.";
    }

    @PostMapping("/logout")
    public String logout(@RequestParam String userId) {
        metricsRegistry.getActiveUsersGauge().decrementAndGet();
        return "User " + userId + " logged out.";
    }
}
```

---

#### **5. Exception Counts**

Capture exception counts by customizing the global exception handler to update metrics.

```java
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@ControllerAdvice
public class MetricsExceptionHandler {
    private final MeterRegistry meterRegistry;

    public MetricsExceptionHandler(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }

    @ExceptionHandler(Exception.class)
    public void handleException(Exception exception) {
        meterRegistry.counter("exceptions.count", "exception.type", exception.getClass().getSimpleName())
                .increment();
    }
}
```

---

### **6. Testing Metrics**

- Start the application and interact with the APIs (e.g., `/users/login`, `/users/logout`, or order processing).
- Access the Prometheus endpoint at `http://localhost:8080/actuator/prometheus`.
- Example metrics exposed:
    
    ```plaintext
    # HELP orders_success_total Number of successfully processed orders
    # TYPE orders_success_total counter
    orders_success_total 5.0
    
    # HELP orders_failure_total Number of failed order processing attempts
    # TYPE orders_failure_total counter
    orders_failure_total 2.0
    
    # HELP orders_processing_time Order processing time in milliseconds
    # TYPE orders_processing_time summary
    orders_processing_time_count 7.0
    orders_processing_time_sum 1500.0
    
    # HELP users_active Number of active users
    # TYPE users_active gauge
    users_active 1.0
    
    # HELP exceptions_count_total Exception counts by type
    # TYPE exceptions_count_total counter
    exceptions_count_total{exception.type="RuntimeException"} 1.0
    ```
    

---

### **Best Practices**

1. **Label Consistency**: Use consistent labels for metrics (e.g., `order.id`, `exception.type`) for easier querying.
2. **Metric Cardinality**: Avoid high cardinality (e.g., metrics with a label for every user) to prevent performance issues.
3. **Alerts and Dashboards**: Use Prometheus to create alerts and Grafana for dashboards based on these metrics.
4. **Leverage Built-in Metrics**: Micrometer integrates with Spring Boot Actuator to provide JVM and application-level metrics.

This implementation offers an extensible and observable foundation for monitoring key application-level metrics in a business application.

The classes `Counter` and `DistributionSummary` used in the code example are from the **Micrometer** library, a popular instrumentation framework for JVM-based applications. Micrometer integrates with monitoring systems like Prometheus, Datadog, Graphite, and others to collect and export metrics.

Here’s a breakdown of these classes:

---

### **1. `Counter`**

- **Purpose**: Tracks the number of times an event has occurred. It is a monotonically increasing value.
- **Use Case**: To count discrete events like:
    - Number of successful transactions (`orderSuccessCounter`).
    - Number of failures (`orderFailureCounter`).

#### Example

- **Library**: Micrometer (`io.micrometer.core.instrument.Counter`).
    
- **Usage**:
    
    ```java
    Counter.builder("orders.success")
           .description("Number of successfully processed orders")
           .register(meterRegistry);
    ```
    
    This creates a counter named `orders.success` that can be incremented each time an order is processed successfully.
    

---

### **2. `DistributionSummary`**

- **Purpose**: Tracks the distribution of a set of values, including:
    - Count.
    - Total sum.
    - Min, max, mean, and percentiles.
- **Use Case**: To record and analyze a range of numerical values, such as:
    - The processing time of orders (`orderProcessingTime`).

#### Example

- **Library**: Micrometer (`io.micrometer.core.instrument.DistributionSummary`).
    
- **Usage**:
    
    ```java
    DistributionSummary.builder("orders.processing.time")
                       .description("Order processing time in milliseconds")
                       .baseUnit("milliseconds")
                       .register(meterRegistry);
    ```
    
    This creates a metric that records individual processing times. Prometheus can then aggregate and analyze these times (e.g., calculating averages or 95th percentile).
    

---

### **3. Library Details**

- **Micrometer** is included in the Spring ecosystem by default through **Spring Boot Actuator**.
- **Relevant Dependencies**: Add these dependencies to your project if not already included:
    
    ```xml
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-registry-prometheus</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    ```
    

---

### **How These Work**

1. **`MeterRegistry`**: A central registry where all metrics (`Counter`, `Gauge`, `Timer`, etc.) are registered. It acts as a bridge between your application and the monitoring backend (e.g., Prometheus).
2. **Creating Metrics**: Metrics like `Counter` or `DistributionSummary` are registered with the `MeterRegistry`.
3. **Recording Data**:
    - For `Counter`: Use `.increment()` to count an event.
    - For `DistributionSummary`: Use `.record(value)` to record individual values (e.g., processing times).

---

### Example Flow

1. **Incrementing Counters**:
    
    ```java
    orderSuccessCounter.increment(); // Each successful order
    orderFailureCounter.increment(); // Each failed order
    ```
    
2. **Recording Distributions**:
    
    ```java
    long startTime = System.currentTimeMillis();
    // Business logic here
    long duration = System.currentTimeMillis() - startTime;
    orderProcessingTime.record(duration); // Record the time taken
    ```
    

---

These classes are part of the instrumentation toolkit that provides precise monitoring and observability in a production-grade application.

Using **OpenTelemetry** for **Application-Level Metrics** involves defining metrics and exporting them to a compatible backend like Prometheus. OpenTelemetry provides powerful tools to record and export metrics while maintaining compatibility with various observability platforms.

Here’s an example of implementing **Application-Level Metrics** using **OpenTelemetry** in a Spring Boot application.

---

### **Implementation Overview**

We will track the following metrics:

1. **Order Creation Metrics**: Success count, failure count, and processing time.
2. **Active Users**: The number of users interacting with the system.

---

### **Dependencies**

Add the OpenTelemetry dependencies to your `pom.xml`:

```xml
<dependency>
    <groupId>io.opentelemetry</groupId>
    <artifactId>opentelemetry-api</artifactId>
</dependency>
<dependency>
    <groupId>io.opentelemetry</groupId>
    <artifactId>opentelemetry-sdk-metrics</artifactId>
    <version>1.29.0</version> <!-- Update with the latest version -->
</dependency>
<dependency>
    <groupId>io.opentelemetry.exporter</groupId>
    <artifactId>opentelemetry-exporter-otlp-metrics</artifactId>
    <version>1.29.0</version>
</dependency>
```

---

### **Code Implementation**

#### **1. Configuring OpenTelemetry**

Create a configuration class to set up the OpenTelemetry SDK and exporters.

```java
import io.opentelemetry.api.GlobalOpenTelemetry;
import io.opentelemetry.api.OpenTelemetry;
import io.opentelemetry.sdk.OpenTelemetrySdk;
import io.opentelemetry.sdk.metrics.SdkMeterProvider;
import io.opentelemetry.sdk.metrics.export.IntervalMetricReader;
import io.opentelemetry.exporter.otlp.metrics.OtlpGrpcMetricExporter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class OpenTelemetryConfig {

    @Bean
    public OpenTelemetry openTelemetry() {
        // Configure OTLP Metric Exporter (e.g., for Prometheus)
        OtlpGrpcMetricExporter exporter = OtlpGrpcMetricExporter.getDefault();

        // Create the MeterProvider and configure the exporter
        SdkMeterProvider meterProvider = SdkMeterProvider.builder()
                .registerMetricReader(IntervalMetricReader.builder()
                        .setMetricExporter(exporter)
                        .setInterval(30, java.util.concurrent.TimeUnit.SECONDS)
                        .build())
                .build();

        OpenTelemetrySdk openTelemetry = OpenTelemetrySdk.builder()
                .setMeterProvider(meterProvider)
                .build();

        GlobalOpenTelemetry.set(openTelemetry);
        return openTelemetry;
    }
}
```

---

#### **2. Metrics Registration**

Use OpenTelemetry’s `Meter` API to define and record metrics.

```java
import io.opentelemetry.api.metrics.Meter;
import io.opentelemetry.api.metrics.MeterProvider;
import io.opentelemetry.api.metrics.LongCounter;
import io.opentelemetry.api.metrics.LongUpDownCounter;
import io.opentelemetry.api.metrics.DoubleHistogram;
import org.springframework.stereotype.Component;

@Component
public class MetricsRegistry {
    private final LongCounter orderSuccessCounter;
    private final LongCounter orderFailureCounter;
    private final DoubleHistogram orderProcessingTime;
    private final LongUpDownCounter activeUsersGauge;

    public MetricsRegistry(OpenTelemetry openTelemetry) {
        MeterProvider meterProvider = openTelemetry.getMeterProvider();
        Meter meter = meterProvider.meterBuilder("application.metrics")
                .setInstrumentationVersion("1.0.0")
                .build();

        // Counter for successful orders
        this.orderSuccessCounter = meter.counterBuilder("orders.success")
                .setDescription("Number of successfully processed orders")
                .setUnit("1")
                .build();

        // Counter for failed orders
        this.orderFailureCounter = meter.counterBuilder("orders.failure")
                .setDescription("Number of failed order processing attempts")
                .setUnit("1")
                .build();

        // Histogram for order processing time
        this.orderProcessingTime = meter.histogramBuilder("orders.processing.time")
                .setDescription("Order processing time in milliseconds")
                .setUnit("ms")
                .build();

        // Gauge for active users
        this.activeUsersGauge = meter.upDownCounterBuilder("users.active")
                .setDescription("Number of active users")
                .setUnit("1")
                .build();
    }

    public void incrementOrderSuccess() {
        orderSuccessCounter.add(1);
    }

    public void incrementOrderFailure() {
        orderFailureCounter.add(1);
    }

    public void recordProcessingTime(double timeMillis) {
        orderProcessingTime.record(timeMillis);
    }

    public void incrementActiveUsers() {
        activeUsersGauge.add(1);
    }

    public void decrementActiveUsers() {
        activeUsersGauge.add(-1);
    }
}
```

---

#### **3. Instrumenting Business Logic**

Use the `MetricsRegistry` component to record metrics during operations.

```java
import org.springframework.stereotype.Service;

@Service
public class OrderService {
    private final MetricsRegistry metricsRegistry;

    public OrderService(MetricsRegistry metricsRegistry) {
        this.metricsRegistry = metricsRegistry;
    }

    public void processOrder(String orderId) {
        long startTime = System.currentTimeMillis();

        try {
            // Simulate order processing logic
            Thread.sleep((long) (Math.random() * 500));

            // Record success
            metricsRegistry.incrementOrderSuccess();
        } catch (Exception e) {
            // Record failure
            metricsRegistry.incrementOrderFailure();
            throw new RuntimeException("Order processing failed", e);
        } finally {
            // Record processing time
            long duration = System.currentTimeMillis() - startTime;
            metricsRegistry.recordProcessingTime(duration);
        }
    }
}
```

---

#### **4. Active Users Gauge**

Track the number of users interacting with the application.

```java
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/users")
public class UserController {
    private final MetricsRegistry metricsRegistry;

    public UserController(MetricsRegistry metricsRegistry) {
        this.metricsRegistry = metricsRegistry;
    }

    @PostMapping("/login")
    public String login(@RequestParam String userId) {
        metricsRegistry.incrementActiveUsers();
        return "User " + userId + " logged in.";
    }

    @PostMapping("/logout")
    public String logout(@RequestParam String userId) {
        metricsRegistry.decrementActiveUsers();
        return "User " + userId + " logged out.";
    }
}
```

---

#### **5. Testing Metrics**

- Start the application and use the APIs to generate metric events (e.g., order processing, user logins).
- Configure **Prometheus** to scrape metrics from the OpenTelemetry collector.
- Example Prometheus metric output:
    
    ```plaintext
    # HELP orders_success Number of successfully processed orders
    # TYPE orders_success counter
    orders_success 5.0
    
    # HELP orders_failure Number of failed order processing attempts
    # TYPE orders_failure counter
    orders_failure 2.0
    
    # HELP orders_processing_time Order processing time in milliseconds
    # TYPE orders_processing_time histogram
    orders_processing_time_count 7.0
    orders_processing_time_sum 1500.0
    
    # HELP users_active Number of active users
    # TYPE users_active gauge
    users_active 3.0
    ```
    

---

### **Benefits of OpenTelemetry**

1. **Standardization**: OpenTelemetry supports standardized metrics, traces, and logs across multiple observability tools.
2. **Flexibility**: Integrates with multiple exporters, including Prometheus, Jaeger, Zipkin, and more.
3. **Rich Ecosystem**: Provides end-to-end observability through metrics, traces, and logs in one framework.

This example demonstrates how to use OpenTelemetry to create application-level metrics with rich instrumentation and flexible backend integration.


In Java, functional programming is achieved through the use of functional interfaces, lambdas, and method references. While the traditional `try-catch-finally` block is an imperative construct, you can use functional programming principles to encapsulate such logic. Below is an example of how to implement the `processOrder` method with functional programming:

---

### **Using Functional Interfaces**

1. **Encapsulate Logic in Functional Interfaces**: Define functional interfaces to handle processing, exceptions, and finalization.
    
2. **Method Implementation**: Replace imperative `try-catch-finally` with a functional approach that separates the concerns of processing, error handling, and final steps.
    

---

### **Code Example**

```java
import java.util.function.Consumer;
import java.util.function.Supplier;

public class OrderService {
    private final MetricsRegistry metricsRegistry;

    public OrderService(MetricsRegistry metricsRegistry) {
        this.metricsRegistry = metricsRegistry;
    }

    public void processOrder(String orderId) {
        executeWithTryCatchFinally(
            () -> simulateOrderProcessing(orderId),      // Main logic
            e -> handleError(e, orderId),                // Error handler
            () -> recordProcessingTime(orderId)          // Final step
        );
    }

    private void simulateOrderProcessing(String orderId) {
        // Simulate processing logic
        System.out.println("Processing order: " + orderId);
        if (Math.random() > 0.5) {
            throw new RuntimeException("Order processing failed!");
        }
        metricsRegistry.incrementOrderSuccess();
    }

    private void handleError(Exception e, String orderId) {
        // Handle the exception
        System.err.println("Error processing order " + orderId + ": " + e.getMessage());
        metricsRegistry.incrementOrderFailure();
    }

    private void recordProcessingTime(String orderId) {
        // Simulate recording processing time
        System.out.println("Recording processing time for order: " + orderId);
        metricsRegistry.recordProcessingTime(Math.random() * 1000);
    }

    private void executeWithTryCatchFinally(Runnable processLogic, Consumer<Exception> errorHandler, Runnable finalizer) {
        try {
            processLogic.run();
        } catch (Exception e) {
            errorHandler.accept(e);
        } finally {
            finalizer.run();
        }
    }
}
```

---

### **Explanation**

1. **`executeWithTryCatchFinally`**:
    
    - This is a reusable method that abstracts the `try-catch-finally` block using functional programming principles.
    - Accepts:
        - A `Runnable` for the main processing logic.
        - A `Consumer<Exception>` for error handling.
        - A `Runnable` for finalization logic.
2. **Encapsulated Logic**:
    
    - **Processing Logic**: `simulateOrderProcessing` simulates the core order processing.
    - **Error Handling**: `handleError` deals with any exceptions thrown during processing.
    - **Finalization**: `recordProcessingTime` executes in the `finally` block.
3. **Functional Composition**:
    
    - Separates concerns for each phase of execution, improving readability and modularity.
    - Encourages reuse of the `executeWithTryCatchFinally` block for other workflows.

---

### **Benefits of This Approach**

- **Reusability**: The `executeWithTryCatchFinally` function is generic and can be reused across different workflows.
- **Separation of Concerns**: Processing, error handling, and finalization are clearly separated into distinct methods.
- **Enhanced Readability**: Each step is encapsulated, making the logic easier to understand.

This approach blends functional programming principles into a traditionally imperative construct, providing a clean and reusable solution.

In reactive programming, try-catch-finally constructs are replaced with operators in the reactive stream pipeline. Here's how you can implement the `processOrder` method using **Reactive Programming** with Project Reactor (a common library for reactive programming in Java).

---

### **Implementation Overview**

1. **Reactive Streams**: Use the `Mono` or `Flux` types from Project Reactor to model asynchronous operations.
    
2. **Error Handling**: Use `.onErrorResume()` or `.onErrorMap()` for handling exceptions in the pipeline.
    
3. **Finalization**: Use `.doFinally()` for cleanup or logging that happens regardless of success or failure.
    

---

### **Reactive `processOrder` Implementation**

```java
import reactor.core.publisher.Mono;
import reactor.util.retry.Retry;
import java.time.Duration;

public class OrderService {
    private final MetricsRegistry metricsRegistry;

    public OrderService(MetricsRegistry metricsRegistry) {
        this.metricsRegistry = metricsRegistry;
    }

    public void processOrder(String orderId) {
        Mono.fromCallable(() -> simulateOrderProcessing(orderId)) // Main logic
            .doOnSuccess(result -> metricsRegistry.incrementOrderSuccess()) // Success handler
            .doOnError(error -> handleError(error, orderId)) // Error handler
            .retryWhen(Retry.fixedDelay(3, Duration.ofSeconds(2))) // Retry logic
            .doFinally(signalType -> recordProcessingTime(orderId)) // Finalizer
            .subscribe(
                success -> System.out.println("Order processed successfully: " + orderId),
                error -> System.err.println("Order failed: " + error.getMessage())
            );
    }

    private String simulateOrderProcessing(String orderId) {
        // Simulate order processing logic
        System.out.println("Processing order: " + orderId);
        if (Math.random() > 0.5) {
            throw new RuntimeException("Order processing failed!");
        }
        return "OrderSuccess";
    }

    private void handleError(Throwable error, String orderId) {
        // Handle the exception
        System.err.println("Error processing order " + orderId + ": " + error.getMessage());
        metricsRegistry.incrementOrderFailure();
    }

    private void recordProcessingTime(String orderId) {
        // Simulate recording processing time
        System.out.println("Recording processing time for order: " + orderId);
        metricsRegistry.recordProcessingTime(Math.random() * 1000);
    }
}
```

---

### **Key Operators**

1. **Pipeline Setup**:
    
    - `Mono.fromCallable`: Wraps synchronous code into a reactive `Mono`.
2. **Error Handling**:
    
    - `.doOnError`: Side-effect operator that handles errors without modifying the stream.
    - `.onErrorResume`: Allows fallback logic or recovery from errors.
3. **Retry Logic**:
    
    - `.retryWhen`: Specifies retry rules (e.g., retry up to 3 times with a 2-second delay).
4. **Finalization**:
    
    - `.doFinally`: Ensures final cleanup happens regardless of success or failure.
5. **Subscription**:
    
    - `.subscribe(success, error)`: Consumes the reactive pipeline, executing the logic.

---

### **Execution Flow**

1. **Processing**:
    
    - The order is processed via `simulateOrderProcessing`.
    - On success, it increments the success counter.
2. **Error Handling**:
    
    - If an error occurs, `handleError` is called, which updates failure metrics.
3. **Retry**:
    
    - If an error occurs, the pipeline retries up to 3 times with a 2-second delay.
4. **Finalization**:
    
    - `recordProcessingTime` is executed at the end, regardless of the pipeline's success or failure.

---

### **Benefits of Reactive Programming**

1. **Non-Blocking**:
    
    - Reactive streams allow processing to happen asynchronously without blocking threads.
2. **Resilience**:
    
    - Retry mechanisms can recover from transient failures.
3. **Separation of Concerns**:
    
    - Each aspect (processing, error handling, finalization) is cleanly separated in the pipeline.
4. **Scalability**:
    
    - Ideal for high-throughput, concurrent environments where resource efficiency is critical.

---

This example demonstrates how to handle the lifecycle of processing, error management, and finalization in a reactive pipeline. It's a powerful way to implement complex workflows in an asynchronous and non-blocking manner.