> By Charu Jain and Sahil Goel

# Overview

In a Kubernetes cluster, depending on the requirements, multiple microservices could be running at any given time. Uniform logging across these microservices is essential for the following reasons:

- **Debugging and troubleshooting:** When an issue occurs in a microservices-based solution, it can be challenging to track down the source of the problem if the logs are not consistent. A uniform logging format makes correlating logs from different services easier, which can help engineers quickly identify and fix the problem.
- **Improved readability**: Consistent logs are easier to read and understand due to uniform data organization.
- **Enhanced analysis**: Consistent logs help perform trend analysis and anomaly detection, allowing the identification of potential problems before they cause outages or other issues.
- **Easier search and filtering:** Consistent logs can be easily searched and filtered by specific fields, making the information we need more accessible. For example, we could search for all logs that contain a particular error message (e.g., HTTP 500) or all logs generated by a specific user.
- **Compliance:** In some industries, such as finance and healthcare, there are strict compliance requirements for logging. A consistent logging format can help organizations comply with these requirements.

Using a structured logging format such as JSON (JavaScript Object Notation) creates uniformity, making it easier to send the logs to a centralized logging solution, and enables a holistic search capability.

# Consistent Logging Across Microservices

Utilizing a standard logging library — added as a dependency to each microservice — is good practice. Standardization enables consistency across the microservices within a Kubernetes cluster, avoids duplication of the configuration or classes in each microservice, and ensures they do not have to maintain a separate logback.xml file.

# LogstashEncoder

The LogstashEncoder is a dependency that can be used to format Spring Boot logs in JSON format, compatible with Logstash. This formatting makes collecting and analyzing Spring Boot logs easier on various platforms, such as Application Insights on AKS (Azure Kubernetes Service).

The Spring Boot LogstashEncoder can be used by adding the following dependency to the Spring Boot project:

<dependency  
   <groupId>net.logstash.logback</groupId>  
   <artifactId>logstash-logback-encoder</artifactId>  
   <version>7.4</version>  
</dependency>  
  
<dependency>  
   <groupId>ch.qos.logback</groupId>  
   <artifactId>logback-classic</artifactId>  
</dependency>

Once the dependency is added, we can configure the LogstashEncoder in the Spring Boot application’s logback.xml configuration file. The following is an example of a logback.xml configuration that uses the LogstashEncoder.

<?xml version="1.0" encoding="UTF-8"?>  
<configuration debug="true">  
    <springProperty scope="context" name="springAppName" source="spring.application.name"/>  
    <springProperty scope="context" name="springAppVersion" source="spring.application.version"/>  
    <contextName>${springAppName}</contextName>  
     
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">  
        <encoder class="net.logstash.logback.encoder.LogstashEncoder" />  
    </appender>  
  
     <root level="DEBUG">  
       <appender-ref ref="console"/>  
     </root>  
</configuration>

With the above configuration, logs will start appearing containing the following fields:

· **@timestamp:** The timestamp of the log event

· **level:** The level of the log event

· **logger_name:** The name of the logger that generated the log event

· **message:** The message of the log event

· **thread_name:** The name of the current thread

· **springAppName:** The name of the Spring Boot application retrieved from application.properties

· **springAppVersion:** The version of the Spring Boot application retrieved from application.properties

· **MDC (Mapped Diagnostic Context):** All MDC parameters set in the application will be output. MDC is used to store contextual information in a log message. With the above logstash configuration, MDC parameters will automatically start appearing in the logs, if any.

For example:

{  
 "@timestamp": "2023-09-11T10:57:59.28808+05:30",  
 "@version": "1",  
 "message": "This is a debug message.",  
 "logger_name": "com.demo.DemoApplication",  
 "thread_name": "main",  
 "level": "DEBUG",  
 "springAppName": "app-customer",  
 "springAppVersion": "0.0.1",  
 "x-session-id": "val"  
}

In this example, “x-session” is an MDC parameter.

# LoggingEventCompositeJsonEncoder

We can customize the format using LoggingEventCompositeJsonEncoder as the encoder instead of LogstashEncoder, as it provides greater flexibility in the JSON format.

LoggingEventCompositeJsonEncoder is composed of one or more JSON providers that contribute to the JSON output. No providers are configured by default in the composite encoders — we must add the ones we want.

Here, as per our project’s requirements, we have configured the encoder:

```xml
<?xml version="1.0" encoding="UTF-8"?  
<configuration debug="true">  
    <springProperty scope="context" name="springAppName" source="spring.application.name"/>  
    <springProperty scope="context" name="springAppVersion" source="spring.application.version"/>  
   <contextName>${springAppName}</contextName>  
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">  
        <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">  
            <providers>  
                <timestamp>  
                    <fieldName>Timestamp/UTC</fieldName>  
                    <timeZone>UTC</timeZone>  
                </timestamp>  
                <logLevel>  
                    <fieldName>Level</fieldName>  
                </logLevel>  
                <threadName>  
                    <fieldName>Thread</fieldName>  
                </threadName>  
                <contextName>  
                    <fieldName>ServiceName</fieldName>  
                </contextName>  
                <pattern>  
                    <!-- the pattern that defines what to include -->  
                    <pattern>  
                        {  
                        "ServiceVersion": "${springAppVersion}",  
                        "ServiceHostName": "${hostname}",  
                        "PID": "${PID:-}"  
                        }  
                    </pattern>  
                </pattern>  
                <mdc/>  
                <arguments>  
                    <includeNonStructuredArguments>true</includeNonStructuredArguments>  
                </arguments>  
                <loggerName>  
                    <fieldName>Logger</fieldName>  
                </loggerName>  
                <callerData>  
                    <classFieldName>Class</classFieldName>  
                    <methodFieldName>Method</methodFieldName>  
                    <fileFieldName>File</fileFieldName>  
                    <lineFieldName>Line</lineFieldName>  
                </callerData>  
                <message>  
                    <fieldName>Message</fieldName>  
                </message>  
                <throwableClassName>  
                    <fieldName>ExceptionClass</fieldName>  
                </throwableClassName>  
                <stackTrace>  
                    <fieldName>StackTrace</fieldName>  
                    <!-- maxLength - limit the length of the stack trace -->  
                    <throwableConverter class="net.logstash.logback.stacktrace.ShortenedThrowableConverter">  
                       <maxDepthPerThrowable>200</maxDepthPerThrowable>  
                        <maxLength>5000</maxLength>  
                        <rootCauseFirst>true</rootCauseFirst>  
                    </throwableConverter>  
                </stackTrace>  
            </providers>  
        </encoder>  
    </appender>  
      
     <root level="DEBUG">  
           <appender-ref ref="console"/>  
      </root>  
</configuration>
```
# Description of appenders


![](https://miro.medium.com/v2/resize:fit:700/1*hrd2kgTt2iE8_Waysj6C1A.png)

**Example**

Here is an example of generated log details with the configuration described above, using LoggingEventCompositeJsonEncoder.

log.atInfo().log("This is an info message.");

For the above logging statement, the application will log the following details:

```xml
{  
 "Timestamp/UTC": "2023-08-10T18:46:24.04716Z",  
 "Level": "INFO",  
 "Thread": "main",  
 "ServiceName": "app-customer",  
 "ServiceVersion": "0.0.1",  
 "ServiceHostName": "WKOZTTXvg3rzP0r",  
 "PID": "23056",  
 "x-session-id": "x-session-id-val",  
 "x-channel-id": "x-channel-val",  
 "Logger": "com.example.demo.DemoApplication",  
 "Class": "com.example.demo.DemoApplication",  
 "Method": "main",  
 "File": "DemoApplication.java",  
 "Line": 23,  
 "Message": "This is an info message."  
}
```

In the above log output, “**x-session-id”** and **“x-channel”** are MDC parameters. They are automatically added to the logs if we have configured the <mdc/> provider in logback.xml.

If we try to generate an “application error” using the following code:

public static void testing() {  
    try {  
            throw new RuntimeException("inside try block", new RuntimeException("testing for error message", new IllegalArgumentException("new illegal msg")));  
        } catch (Exception e) {  
            log.error(e.getMessage(), e);  
        }  
}

The log details will look like this:

```xml
{  
 "Timestamp/UTC": "2023-08-10T18:46:24.0691734Z",  
 "Level": "ERROR",  
 "Thread": "main",  
 "ServiceName": "app-customer",  
 "ServiceVersion": "0.0.1",  
 "ServiceHostName": "WKJZTTXvg1rzQ0r",  
 "PID": "23056",  
 "x-session-id": "x-session-id-val",  
 "x-channel-id": "x-channel-val",  
 "Logger": "com.example.demo.DemoApplication",  
 "Class": "com.example.demo.DemoApplication",  
 "Method": "testing",  
 "File": "DemoApplication.java",  
 "Line": 38,  
 "Message": "inside try block",  
 "ExceptionClass": "RuntimeException",  
 "StackTrace": "java.lang.IllegalArgumentException: new illegal msg\r\n\t... 2 common frames omitted  
                \r\nWrapped by: java.lang.RuntimeException: testing for error message\r\n\t...   
                2 common frames omitted\r\nWrapped by: java.lang.RuntimeException: inside try block\r\n\tat com.example.demo.DemoApplication.testing(DemoApplication.java:36)\r\n\tat com.example.demo.DemoApplication.main(DemoApplication.java:30)\r\n"  
}
```

# Masking

We need to mask logs to comply with data security and privacy requirements.

In this approach, each microservice does not have a separate logback.xml file to specify the masking configuration. We need a framework in which each microservice can pass the masking requirements to the shared logging library. In our approach, each microservice defines the masking patterns in its respective application configuration, automatically modifying the logback.xml behavior based on the supplied configuration.

To enable this, we need to embed MaskingJsonGeneratorDecorator inside LoggingEventCompositeJsonEncoder.

We also need to write a custom valueMasker for this, for example:

```xml
<?xml version="1.0" encoding="UTF-8"?  
<configuration debug="true">  
    <springProperty scope="context" name="springAppName" source="spring.application.name"/>  
    <springProperty scope="context" name="springAppVersion" source="spring.application.version"/>  
    <springProperty scope="context" name="maskingEnabled" source="log.masking.enabled"/>  
    <springProperty scope="context" name="maskPattern" source="log.masking.maskPattern"/>  
      
    <contextName>${springAppName}</contextName>  
    <appender name="console_masking" class="ch.qos.logback.core.ConsoleAppender">  
          <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">  
             <jsonGeneratorDecorator class="net.logstash.logback.mask.MaskingJsonGeneratorDecorator">  
                 <valueMasker class="com.example.demo.CustomMaskingJsonGeneratorDecorator">  
                     <customRegex>${maskPattern}</customRegex>  
                 </valueMasker>  
                  <path>Message/*</path>  
              </jsonGeneratorDecorator>  
              <providers>  
                  <timestamp>  
                      .........  
                  </timestamp>  
                   // same as described above  
              </providers>  
          </encoder>  
      </appender>  
    
      <appender name="console" class="ch.qos.logback.core.ConsoleAppender">  
          <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">  
              <providers>  
                  <timestamp>  
                      <fieldName>Timestamp/UTC</fieldName>  
                      <timeZone>UTC</timeZone>  
                  </timestamp>  
                   // same as described above  
              </providers>  
          </encoder>  
      </appender>  
    
      <if condition='property("maskingEnabled").equals("true")'>  
          <then>  
              <root level="DEBUG">  
                  <appender-ref ref="console_masking"/>  
              </root>  
          </then>  
          <else>  
              <root level="DEBUG">  
                  <appender-ref ref="console"/>  
              </root>  
          </else>  
      </if>  
  </configuration>  
```

Please note the condition for activating the appender-ref.

# CustomMaskingJsonGeneratorDecorator

@Slf4j  
public class CustomMaskingJsonGeneratorDecorator implements ValueMasker {  
      
    private String customRegex;  
  
    public CustomMaskingJsonGeneratorDecorator() {}  
  
    public String getCustomRegex() {  
        return customRegex;  
    }  
  
    public void setCustomRegex(String customRegex) {  
        this.customRegex = customRegex;   
             // This will set it with customRegex property set in logback xml  
       }  
  
@Override       
public Object mask(JsonStreamContext context, Object value) {  
        // add your custom masking implementation          
          if (value instanceof CharSequence) {              
              return maskMessage((String) value);          
          }  
        return value;  
    }     
}

In the example above, the masking pattern is provided by each microservices application.properties. The “log.masking.maskPattern” property contains mask pattern.

Our CustomMaskingJsonGeneratorDecorator class implements ValueMasker interface. This provides Override method mask() which helps in masking.

Mask method accepts two parameters: context and value, referring to the log property and its value.

**Example**

For the following logging statement,

log.info("Some message with \"address\":{\"code\":\"abcd\",\"locality\":\"xyzd\",\"street\":\"j Strervsjkd ,nsjdsy\"},\"ssn\": \"anbgdb123\"");

…the output will be:

```xml
{  
 "Timestamp/UTC": "2023-08-10T19:04:48.8361333Z",  
 "Level": "INFO",  
 "Thread": "main",  
 "ServiceName": "app-customer",  
 "ServiceVersion": "0.0.1",  
 "ServiceHostName": "WKWZTTXvg6rzT0r",  
 "PID": "30576",  
 "Session": "x-session-id-val",  
 "Channel": "x-channel-val",  
 "Logger": "com.example.demo.DemoApplication",  
 "Class": "com.example.demo.DemoApplication",  
 "Method": "main",  
 "File": "DemoApplication.java",  
 "Line": 26,  
 "Message": "Some message with \"address\":*****************************************************************,  
            \"ssn\": \"*********\""  
}
```

# Conclusion

Consistent logging by multiple microservices sharing a common Kubernetes cluster improves troubleshooting, reliability and security. Additionally, having a common logging library for multiple microservices removes duplication by utilizing a single copy logback.xml at the library level.

So, if you’re not already utilizing consistent logging, we encourage you to consider adopting the same approach.

We hope you found this useful.

For more details on the logstash encoder, check out: [https://github.com/liangyanfeng/logstash-logback](https://github.com/liangyanfeng/logstash-logback) encoder/blob/master/README.md

# About Publicis Sapient

Publicis Sapient is a digital business transformation partner that helps organizations get to their future, digitally enabled state, both in the way they work and the way they serve their customers.

For more than 30 years, we’ve helped some of the world’s biggest organizations build a competitive advantage through digital.

[Visit our website](https://www.publicissapient.com/) to find out more.