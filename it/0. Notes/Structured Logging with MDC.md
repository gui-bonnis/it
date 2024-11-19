How to create a structure log with MDC, builder pattern, logback and springboot, in order to minimize verbosity but have all log levels when needed and all informations from the business processing in any point in time

###### ChatGPT said:

ChatGPT

Memory updated

To create a structured logging mechanism in a Spring Boot application using MDC (Mapped Diagnostic Context), the builder pattern, and Logback, while minimizing verbosity, you can follow these steps:

### 1. **Setup Logback for Structured Logging**

- Define a custom Logback configuration (`logback-spring.xml` or `logback.xml`).

xml

Copy code

`<configuration>     <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">         <encoder>             <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>             <!-- Use JSON format for structured logs -->             <json>                 <timestampPattern>yyyy-MM-dd HH:mm:ss</timestampPattern>                 <messagePattern>%msg</messagePattern>                 <mdcPattern>%mdc</mdcPattern>             </json>         </encoder>     </appender>      <root level="INFO">         <appender-ref ref="CONSOLE" />     </root> </configuration>`

- You can also configure JSON logging libraries like Logstash to further structure your logs.

### 2. **Use MDC for Contextual Information**

- MDC allows you to add context information to your logs that will automatically be included in every log message.

java

Copy code

`import org.slf4j.MDC;  public class BusinessService {     public void process(String transactionId, String userId) {         try {             MDC.put("transactionId", transactionId);             MDC.put("userId", userId);                          // Business logic             logger.info("Processing started");                      } finally {             MDC.clear();         }     } }`

### 3. **Implement the Builder Pattern for Logging**

- To minimize verbosity and control log levels, you can use the builder pattern to construct log messages.

java

Copy code

`public class LogBuilder {     private final Map<String, Object> logData = new HashMap<>();          public LogBuilder add(String key, Object value) {         logData.put(key, value);         return this;     }      public void log(Level level, String message) {         // Convert logData to JSON or another structured format         String structuredMessage = convertToStructuredFormat(logData);         switch (level) {             case INFO:                 logger.info("{} - {}", message, structuredMessage);                 break;             case DEBUG:                 logger.debug("{} - {}", message, structuredMessage);                 break;             // Handle other log levels as needed             default:                 logger.info("{} - {}", message, structuredMessage);                 break;         }     }      private String convertToStructuredFormat(Map<String, Object> logData) {         // Convert the log data to a structured format, e.g., JSON         return new Gson().toJson(logData);     } }`

- Use the `LogBuilder` to create logs with all necessary business processing information.

java

Copy code

`public class BusinessService {     private final LogBuilder logBuilder = new LogBuilder();      public void process(String transactionId, String userId) {         MDC.put("transactionId", transactionId);         MDC.put("userId", userId);         try {             logBuilder.add("transactionId", transactionId)                       .add("userId", userId)                       .log(Level.INFO, "Processing started");              // Business logic              logBuilder.log(Level.DEBUG, "Processing step 1 completed");          } finally {             MDC.clear();         }     } }`

### 4. **Control Verbosity with Log Levels**

- Configure your logback settings to control verbosity and enable different log levels as needed in various environments (e.g., DEBUG in development, INFO in production).
    
- You can adjust log levels programmatically if needed:
    

java

Copy code

`Logger rootLogger = (Logger) LoggerFactory.getLogger(Logger.ROOT_LOGGER_NAME); rootLogger.setLevel(Level.DEBUG);`

### 5. **Include Contextual Information at Any Point in Time**

- Use MDC and the builder pattern to ensure that all relevant business information is included in your logs at any point in time.

### 6. **Minimize Verbosity**

- By using the builder pattern and structured logging, you can ensure that logs are only as verbose as necessary, containing all critical information without being overly wordy.

This setup will allow you to create detailed, structured logs with all the necessary information while keeping your logging configuration and code clean and maintainable.