To generate nested JSON output using Logback, you need to use a library like **Logstash Logback Encoder**, which provides features for structured and nested JSON logging. By default, if your JSON output is flattened, it might be due to the way you are configuring your log structure.

Here’s how you can configure Logback to produce nested JSON logs:

### 1. Add Logstash Logback Encoder Dependency

Ensure you include the Logstash Logback Encoder dependency in your project:

**For Maven:**

```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
```

**For Gradle:**

```gradle
implementation 'net.logstash.logback:logstash-logback-encoder:7.4'
```

### 2. Configure `logback-spring.xml` (or `logback.xml`)

Use the `JsonLayout` or `CompositeJsonEncoder` to structure your logs with nested JSON:

#### Example with Nested JSON

```xml
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <includeContext>true</includeContext>
            <includeMdc>true</includeMdc>
            <customFields>{"application":"my-app"}</customFields>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE" />
    </root>
</configuration>
```

### 3. Nest Data Using MDC or Structured Arguments

You can use **MDC** (Mapped Diagnostic Context) or structured arguments in your application to generate nested JSON.

#### Adding MDC for Nested Data

```java
import org.slf4j.MDC;

public void logWithNestedJson() {
    MDC.put("user", "{\"id\":123,\"name\":\"John Doe\"}");
    logger.info("Processing request");
    MDC.clear(); // Clear MDC after logging
}
```

#### Using Structured Arguments (Logstash Encoder supports this natively):

```java
import net.logstash.logback.argument.StructuredArguments;

public void logWithStructuredArgs() {
    logger.info("Order processed",
        StructuredArguments.keyValue("order", Map.of("id", 123, "amount", 99.99))
    );
}
```

### 4. Verify Nested JSON Output

When properly configured, logs will appear with nested JSON structures like:

```json
{
  "timestamp": "2024-12-05T12:34:56.789",
  "level": "INFO",
  "logger": "com.example.MyClass",
  "message": "Order processed",
  "order": {
    "id": 123,
    "amount": 99.99
  },
  "application": "my-app"
}
```

### Common Issues to Check

- **Flattened JSON**: Check if you are using `MDC` keys that aren't formatted as JSON. Logstash Encoder won't automatically parse a plain string into a nested object.
- **Dependencies**: Ensure that the correct version of Logstash Encoder is added.
- **Spring Boot Configuration**: If using `logback-spring.xml`, avoid using `logging.file` or `logging.pattern.*` in `application.properties` as they might override the custom layout.

This approach ensures your logs are neatly nested in JSON format.

Yes, it is possible to generate the same kind of nested JSON output using `net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder`. This encoder provides a more modular and flexible way to configure JSON output by allowing you to define individual components (`providers`) for each part of the log event.

Here’s how you can configure `LoggingEventCompositeJsonEncoder` to produce nested JSON output:

---

### Example Configuration Using `LoggingEventCompositeJsonEncoder`

```xml
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <providers>
                <!-- Timestamp -->
                <timestamp>
                    <fieldName>timestamp</fieldName>
                </timestamp>

                <!-- Log level -->
                <logLevel>
                    <fieldName>level</fieldName>
                </logLevel>

                <!-- Logger name -->
                <loggerName>
                    <fieldName>logger</fieldName>
                </loggerName>

                <!-- Message -->
                <message>
                    <fieldName>message</fieldName>
                </message>

                <!-- Custom fields -->
                <customFields>
                    <customFields>
                        {"application": "my-app"}
                    </customFields>
                </customFields>

                <!-- MDC (for contextual/nested data) -->
                <mdc>
                    <fieldName>context</fieldName>
                </mdc>

                <!-- Thread name -->
                <threadName>
                    <fieldName>thread</fieldName>
                </threadName>
            </providers>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE" />
    </root>
</configuration>
```

---

### Adding Nested Data via MDC

You can add custom fields or nested data dynamically in your application by using MDC (Mapped Diagnostic Context). Logback will automatically include these values if `mdc` is configured in the encoder.

#### Example:

```java
import org.slf4j.MDC;

public class LoggingExample {
    public static void main(String[] args) {
        MDC.put("user", "{\"id\":123,\"name\":\"John Doe\"}");
        MDC.put("order", "{\"id\":456,\"total\":78.90}");
        logger.info("Processing request");
        MDC.clear();
    }
}
```

---

### Expected Output

With the above configuration and MDC values, you should get logs in the following format:

```json
{
  "timestamp": "2024-12-05T12:34:56.789",
  "level": "INFO",
  "logger": "com.example.LoggingExample",
  "message": "Processing request",
  "application": "my-app",
  "context": {
    "user": {
      "id": 123,
      "name": "John Doe"
    },
    "order": {
      "id": 456,
      "total": 78.90
    }
  },
  "thread": "main"
}
```

---

### Key Advantages of `LoggingEventCompositeJsonEncoder`

1. **Flexible Structure**: You can fully customize the log structure by defining multiple providers for various parts of the log.
2. **Custom Fields**: Easily include static or dynamic custom fields.
3. **Nested MDC Values**: Automatically supports nested JSON if MDC values are in JSON format.

---

### Notes

- Ensure MDC values are well-formed JSON strings if you want them to appear as nested objects in the logs.
- You can extend functionality by writing custom `JsonProviders` if you have specific requirements for log fields.
- Avoid redundant configurations (e.g., using both `LoggingEventCompositeJsonEncoder` and `LogstashEncoder` in the same appender).

This approach provides complete control over your log structure and ensures that you can generate clean, nested JSON logs.