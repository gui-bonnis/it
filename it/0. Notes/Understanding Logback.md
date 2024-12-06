## Why am I writing this to you?

Two years ago I made the decision to immerse myself fully in the world of software development and it has been a process in which I have discovered a world full of knowledge to explore. Faced with this, I have decided to choose various topics and apply the following process to each one: research, practical application, and finally try to explain what I have learned in an article. Therefore, this article is the result of said process applied to the topic this article deals with.

## What will you find in this article?

In this article, I try to explain the logging system of a Spring-boot/Kotlin application that uses Logback as a logging framework. Analyzing how our application and the logging system are configured and the flow of a request to log a message from the moment it is executed until it reaches its destination. In the context of Logback and for our study case, that destination will be “System.out”(stdout).

## Where I started

By the time I started going deep into this subject, the only things I knew were:

- How to make a logging request at any point in the code.  
- View the logs in Kibana UI

So let’s start with the logging request. As we see in the following code block

import org.slf4j.LoggerFactory.getLogger  
import org.springframework.http.ResponseEntity  
import org.springframework.web.bind.annotation.GetMapping  
import org.springframework.web.bind.annotation.RestController  
  
@RestController  
class GetController {  
  
    companion object {  
        private val logger = getLogger(GetController::class.java)  
    }  
  
    @GetMapping("/log")  
    fun doSomething(): ResponseEntity<String> {  
        val response = "logs should be as wanted"  
        logger.info("logging a log from {}", GetController::class.simpleName)  
        return ResponseEntity.ok(response)  
    }  
}

The process begins by getting a logger with the name “GetController”. This logger is used for logging the message “_logging a log from GetController_”.

As we see in the imports of the class file **GetController**, we depend only on the SLF4J library to achieve the logging action. The Simple Logging Facade for Java (SLF4J) permits the use of any logging framework under its abstraction layer, such as java.util.logging, log4j, reload4j and logback. In our case, we are going to use Logback.

Our study case is a spring-boot application, which uses [Commons Logging](https://commons.apache.org/logging) for all internal logging but leaves the underlying log implementation open. Default configurations are provided for [Java Util Logging](https://docs.oracle.com/javase/8/docs/api/java/util/logging/package-summary.html), [Log4J2](https://logging.apache.org/log4j/2.x/), and [Logback](https://logback.qos.ch/). By default, with the use of “Spring-boot-starters”, Logback is used for logging, so Logback will be analyzed in this article.

# Setting Spring-Boot App to use Logback

For now, we know that Logback will be our logging framework, but how can we configure it to comply with our requirements?

Logback is divided into three modules: logback-core, logback-classic, and logback-access.  
The _core_ module lays the groundwork for the other two modules.  
The _classic_ module extends _core and_ corresponds to a significantly improved version of log4j. Logback-classic natively implements the [SLF4J API](http://www.slf4j.org/).  
The third module, _access_, integrates with Servlet containers to provide HTTP-access logging functionality.

In addition, the classic module is used in the context of our application, and because of that, we will say that “Application logs” are its responsibility. Conversely, the logback-access module is used in the context of servlet containers; therefore, “Access logs” are its responsibility. Because of this separation of concerns, both context needs to be configured separately.

## **Application-logs**

The simplest way to use Logback Classic (HTTP access logs excluded) is through the starters, which all depend on `spring-boot-starter-logging`. A web application needs only `spring-boot-starter-web` since it depends on the logging starter transitively. So we need to add the following dependency in our “_build.gradle.kts”_ file:

implementation("org.springframework.boot:spring-boot-starter-web")

**Spring-boot default configuration**

The following files are provided under `org/springframework/boot/logging/logback/`:

- `defaults.xml` - Provides conversion rules, pattern properties, and common logger configurations.
- `console-appender.xml` - Adds a `ConsoleAppender` named `CONSOLE` that uses the `CONSOLE_LOG_PATTERN` to format the message.
- `file-appender.xml` - Adds a `RollingFileAppender` named `FILE` using the `FILE_LOG_PATTERN` and `ROLLING_FILE_NAME_PATTERN` with appropriate settings.
- a legacy `base.xml` file is provided for compatibility with earlier versions of Spring Boot.

By default the LOG_FILE is defined in the `base.xml` file as

`<property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/spring.log}"/>`

This configuration sets by default the creation of a console appender and a file appender with a predefined rolling policy, file name, and path as shown before. Why these files are included and the appender concept will be treated later in this article.

## Acces-logs

By default, the access logs aren’t enabled. We can enable them by adding a property to “_application.yaml” file_:

server.tomcat.accesslog.enabled: true

but this configuration only allows to use default logging format and storing log events to a file in a temporary directory.

To enable logback-classic functionality but in the scope of HTTP access logging, we need to use the logback-access module that integrates with Servlet containers such as Jetty or Tomcat. The easiest way to configure logback-acces in our Spring-Boot application is to use the [**logback-access-spring-boot-starter**](https://github.com/akkinoc/logback-access-spring-boot-starter) developed by [**akkinok**](https://github.com/akkinoc/akkinoc) adding the following dependency.

implementation("dev.akkinoc.spring.boot:logback-access-spring-boot-starter")

and set the following properties

logback.access:  
    enabled: true  
    config: "classpath:logback-access-spring.xml"  
    local-port-strategy: local  
  
#To see the full list of property options go to   
# https://github.com/akkinoc/logback-access-spring-boot-starter

Another option to configure Logback-acces instead of using the starter is adding a @Configuration class to add LogbackValve into the embedded Tomcat. In which, we define the configuration-file to be used to configure our access logs.

> _What is a Valve? In the official Apache Tomcat documentation we read: ”A_ **_Valve_** _element represents a component that will be inserted into the request processing pipeline for the associated Catalina container (_[_Engine_](https://tomcat.apache.org/tomcat-7.0-doc/config/engine.html)_,_ [_Host_](https://tomcat.apache.org/tomcat-7.0-doc/config/host.html)_, or_ [_Context_](https://tomcat.apache.org/tomcat-7.0-doc/config/context.html)_). Individual Valves have distinct processing capabilities”_

import ch.qos.logback.access.tomcat.LogbackValve  
import org.springframework.boot.web.embedded.tomcat.TomcatContextCustomizer  
import org.springframework.context.annotation.Bean  
import org.springframework.context.annotation.Configuration  
import java.nio.file.Files  
  
@Configuration  
open class AccessLogConfiguration {  
  
    @Bean  
    fun addLogbackAccessValve() = TomcatContextCustomizer { context ->  
        
      javaClass.getResourceAsStream("/our-logback-access-config-file-name.xml")  
         .use {  
            Files.createDirectories((context.catalinaBase.toPath()  
                .resolve(LogbackValve.DEFAULT_CONFIG_FILE)).parent)  
            Files.copy(it, context.catalinaBase.toPath()  
                .resolve(LogbackValve.DEFAULT_CONFIG_FILE))  
        }  
        LogbackValve().let {  
            it.isQuiet = true  
            context.pipeline.addValve(it)  
        }  
    }  
}

# Logback Configuration

Once we have the dependencies needed to set logback as our logging framework, we’ll see how our Logback instance is configured to process and output our application logs with the rules and format we want.

For the sake of simplicity, let's avoid Logback configuration process which is described in the [Lockback Oficial Manual](https://logback.qos.ch/manual/configuration.html). We’ll just point out that at application start-up, Logback will auto-configure itself and will look for logback-spring.xml and logback-access-spring.xml configuration files located in the “_src/main/resources_” package, as it’s shown in the following image. Just to mention, configuration files like logback.xml and logback-access.xml would work too, but Spring-Boot includes a number of extensions to Logback that can help with advanced configuration and to use these extensions the adding of “_spring_” to the configuration file names is needed. You can check the options these extensions enable in this [link](https://docs.spring.io/spring-boot/docs/2.3.9.RELEASE/reference/html/spring-boot-features.html#boot-features-logback-extensions).

Let's dive into configuration files and their components to explain some architectural characteristics of Logback. Here you can see an example:

<xml version="1.0" encoding="UTF-8">  
<configuration>  
 <include resource="org/springframework/boot/logging/logback/defaults.xml"/>  
 <include resource="org/springframework/boot/logging/logback/console-appender.xml"/>  
 <springProperty name="team" scope="context" source="logging.ownerteam" defaultValue="-"/>  
   
  <springProfile name="dev">  
    <springProperty name="consoleLogLevel" source="logging.console.level" defaultValue="INFO"/>  
    <appender name="ASYNC_CONSOLE" class="ch.qos.logback.classic.AsyncAppender">  
      <neverBlock>true</neverBlock>  
      <appender-ref ref="CONSOLE"/>  
    </appender>  
    <root level="${consoleLogLevel}">  
      <appender-ref ref="ASYNC_CONSOLE"/>  
    </root>  
  </springProfile>  
    
  <springProfile name="k8s">  
    <appender name="CONSOLE-JSON" class="ch.qos.logback.core.ConsoleAppender">  
      <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">  
        <providers>  
          <arguments>  
            <fieldName>context</fieldName>  
          </arguments>  
          <pattern>  
            <pattern>  
              {  
                "date": "%date{ISO8601,GMT+2}",  
                "priority": "%level",  
                "pid": "${PID:- }",  
                "thread": "%t",  
                "logger": "%logger",  
                "message": "%.-10000msg",  
                "stacktrace": "%.-10000throwable",  
                "team": "%property{team}"  
               }  
            </pattern>  
          </pattern>  
          <mdc>net.logstash.logback.composite.loggingevent.MdcJsonProvider</mdc>  
        </providers>  
      </encoder>  
    </appender>  
    <appender name="ASYNC_CONSOLE_JSON" class="ch.qos.logback.classic.AsyncAppender">  
      <neverBlock>true</neverBlock>  
      <appender-ref ref="CONSOLE-JSON"/>  
    </appender>  
    <logger name="com.facuramallo" level="INFO" />  
    <root level="INFO">  
      <appender-ref ref="ASYNC_CONSOLE_JSON"/>  
    </root>  
  </springProfile>  
</configuration>

As mentioned before, the adding of “spring” to the name of the configuration files enables some spring-boot extensions that we can see in image above:

- The `<springProfile name="dev">` element allows defining different logging configurations for different profiles. The configuration will be applied depending on which profile is defined through the properties `spring.profiles.active: "dev"` set in the `application.yml` file.

The “**springProperty”** tag, allows us to define and use application properties, We can see two examples:

- `<springProperty name="consoleLogLevel" source="logging.console.level" defaultValue="INFO"/>` → it takes de value from the `logging.console.level` property in the `application.yml` file or defaults to `“INFO”`.
- `<springProperty name="team" scope="context" source="logging.ownerteam" defaultValue="-"/>` → it takes the value from the `logging.ownerteam` property in the `application.yml` file or defaults to `“-”`.

Following the hierarchy outside-in, the next elements are appenders, logger, and the root tag. To understand this structure let's explain some Logback basics.

Logback is built upon three main classes: `Logger`, `Appender` and `Layout`. The `Logger` class is part of the logback-classic module. On the other hand, the `Appender` and `Layout` interfaces are part of logback-core.

## Loggers

Loggers are named entities. Their names are case-sensitive and they follow the hierarchical naming rule. Every single logger is attached to a `LoggerContext` which is responsible for manufacturing loggers as well as arranging them in a tree-like hierarchy.

Loggers may be assigned a level, if not, it inherits one from its closest ancestor with an assigned level.

> _The effective level for a given logger_ L_, is equal to the first non-null level in its hierarchy, starting at_ L _itself and proceeding upwards in the hierarchy towards the root logger._

In the configuration file example, loggers are defined by the `<logger name="com.facuramallo" level="INFO" />` element where we set a name to the logger and the effective level.

The **_ROOT logger_** has its special tag because it resides at the top of the logger hierarchy and is exceptional because it is part of every hierarchy at its inception.

<root level="INFO">  
  <appender-ref ref="ASYNC_CONSOLE_JSON"/>  
</root>

Here we are defining the root logger level and attaching to it the `"ASYNC_CONSOLE_JSON"` appender. By attaching an appender to a logger, we assign an output destination for that logger's logging requests. Many appenders can be attached to a logger. In this example, we tell the root logger to write logs to the console.

## Loggers basic selection rule

> _The first and foremost advantage of any logging API over plain_ `_System.out.println_` _resides in its ability to disable certain log statements while allowing others to print unhindered. This capability assumes that the logging space, that is, the area of all possible logging statements, is categorised according to some developer-chosen criteria._

From the block above we can say that in the case of logback, the developer-chosen criteria is defined by the logger's effective level and a basic selection rule that states that a log request of level _p_ issued to a logger having an effective level _q_, is enabled if _p >= q_.

This rule is at the heart of logback. It assumes that levels are ordered as follows: `TRACE < DEBUG < INFO < WARN < ERROR` and is directly related with the basic printing methods in the Logger interface, that are:

package org.slf4j;   
public interface Logger {  
// Printing methods:   
  public void trace(String message);  
  public void debug(String message);  
  public void info(String message);   
  public void warn(String message);   
  public void error(String message);   
}

In the configuration file shown above, you can see the definition of the level of the Logger `"com.facuramallo"` to “INFO”, meaning that every logger defined as a child of it, will have an effective level “INFO” and therefore any logging request with level “DEBUG” or “TRACE” would be discarded. In other words, every Logger defined in a class under the main package “com.facuramallo” will have an “INFO” effective level.

## Appenders

Logback delegates the task of writing a logging event to named entities called appenders and allows _logging requests_ to print to multiple destinations. In logback speak, an output destination is called an appender. Currently, appenders exist for the console, files, remote socket servers, MySQL, PostgreSQL, Oracle and other databases, JMS, and remote UNIX Syslog daemons.

The main method of the Appender interface, **doAppend()**, takes an object of type E as its only parameter. The actual type of E will vary depending on the logback module. Within the logback-classic module, E would be of type **_ILoggingEvent_**, and within the logback-access module, it would be of type **_IAccessEvent_**.

> **_The doAppend() method is perhaps the most important in the logback framework. It is responsible for outputting the logging events in a suitable format to the appropriate output device_**_._

In our example configuration file, we set different appenders configuration by profile:

**_Profile “dev”_**

<appender name="ASYNC_CONSOLE" class="ch.qos.logback.classic.AsyncAppender">  
    <neverBlock>true</neverBlock>  
    <appender-ref ref="CONSOLE"/>  
</appender>

Here the appender named “CONSOLE” is attached to the “ASYNC_CONSOLE” appender of type _AsyncAppender.  
AsyncAppender_ acts as a dispatcher to another appender. It buffers **_ILoggingEvents_** and dispatches them to another appender asynchronously. This improves the application’s performance because it allows it not to have to wait for the logging subsystem to complete the action, hence, non-blocking the application.

The “CONSOLE” appender is defined in the file `"org/springframework/boot/logging/logback/console-appender.xml"` and is part of the default logging configuration provided by spring-boot. We add this file using the `<include>` element. `<include resource="org/springframework/boot/logging/logback/console-appender.xml"/>`

If we take a look at this file

<xml version="1.0" encoding="UTF-8">  
<included>  
  <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">  
    <encoder>  
      <pattern>${CONSOLE_LOG_PATTERN}</pattern>  
      <charset>${CONSOLE_LOG_CHARSET}</charset>  
    </encoder>  
  </appender>  
</included>

An appender with name “CONSOLE” is defined with an encoder with a predefined pattern and charset.`${CONSOLE_LOG_PATTERN}` and `${CONSOLE_LOG_CHARSET}` are properties defined in the file we include as

<include resource="org/springframework/boot/logging/logback/defaults.xml"/> 

where spring-boot define another default properties as well.

So, with this configuration, we use spring-boot default logging configuration for the “dev” profile.

**_Profile “k8s”_**

This profile is intended for a production environment, so we changed some things:

- an async appender is also used to avoid application blocking, in this case, we attach `"CONSOLE-JSON"` appender
- for the `"CONSOLE-JSON"` appender we change the encoder in order to achieve JSON formatted logs. We do this because is easier for logging aggregation systems to manage JSON objects. In a more technical language, we intend to use **structured logging** practice.
- to achieve structured logging we use the library [**logstash-logback-encoder**](https://github.com/logfellow/logstash-logback-encoder) which provides logback encoders, layouts, and appenders to log in JSON and other formats supported by Jackson. In this particular case we use the more flexible option, that is **_LoggingEventCompositeJsonEncoder_**, please refer to [_logstash-logback-encoder#composite-encoderlayout_](https://github.com/logfellow/logstash-logback-encoder#composite-encoderlayout) for a full specification.

## Encoders

Encoders are responsible for transforming an incoming event into a byte array and writing out the resulting byte-array onto the appropriate `OutputStream`.

In our configuration file, a “CONSOLE_JSON” appender is set with an encoder of type `LoggingEventCompositeJsonEncoder`. When this encoder is used, it’s mandatory to define the providers we want to use.

<appender name="CONSOLE-JSON" class="ch.qos.logback.core.ConsoleAppender">  
  <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">  
    <providers>  
        <arguments>  
            <fieldName>context</fieldName>  
        </arguments>  
        <pattern>  
          <pattern>  
              {  
              "date": "%date{ISO8601,GMT+2}",  
              "priority": "%level",  
              "pid": "${PID:- }",  
              "thread": "%t",  
              "logger": "%logger",  
              "message": "%.-10000msg",  
              "stacktrace": "%.-10000throwable",  
              "team": "%property{team}"  
              }  
          </pattern>  
        </pattern>  
        <mdc>net.logstash.logback.composite.loggingevent.MdcJsonProvider</mdc>  
    </providers>  
  </encoder>  
</appender>

For this configuration, we use:

- Arguments provider → Outputs fields from the event arguments array that are defined by [**Structured Arguments**](https://github.com/logfellow/logstash-logback-encoder#event-specific-custom-fields)**.** Adding the `<fieldName>context</fieldName>` element will add the field “context” with its value into the JSON output
- pattern provider → Outputs fields from a configured JSON Object string, substituting patterns supported by logback’s **_PatternLayout_**. Refer to official documentation about [**_PatternLayout_**](https://logback.qos.ch/manual/layouts.html#ClassicPatternLayout) and [**_Pattern JSON Provider_**](https://github.com/logfellow/logstash-logback-encoder#pattern-json-provider).
- mdc provider → defined by the class “_MDCJsonProvider_”. Outputs entries from the [Mapped Diagnostic Context (MDC).](https://logback.qos.ch/manual/mdc.html)

## Layouts

Layouts are logback components responsible for transforming an incoming event into a String. The `format()` method in the `[Layout](https://logback.qos.ch/xref/ch/qos/logback/core/Layout.html)` interface takes an object that represents an event (of any type) and returns a String. In the context of modern apps, layouts are no longer used because the approach to append logs to files has change toward logging aggregation systems in containerized apps into clusters or cloud environments where logs are delivered as byte array streams.

## Conclusion

Summarising, Layouts and Encoders are responsible for formatting and transforming a logging event into a byte array that Appenders will output to the destination it represents when a Logger emits a logging event.

For now, we follow the flow from the logging request executed by a logger to its format and finally how it ends up reaching its final output destination. This task is the responsibility of our application context. But what happens next? I work in a developer environment where application and access logs are accessible from a Kibana UI. From deductive thinking, we can say that for that to happen, logs need to be persisted and accessible for a Kibana instance to show my query result in the display. I will work out this second part in a [different article](https://medium.com/@facuramallo8/logging-aggregation-system-d94f60f92dd0).

Thanks for reading, hope it’s useful!!

**Github Repository :** [https://github.com/FacuRamallo/Logging-Agregation-System-ELKK](https://github.com/FacuRamallo/Logging-Agregation-System-ELKK)

**Further reading**: [https://medium.com/@facuramallo8/logging-aggregation-system-d94f60f92dd0](https://medium.com/@facuramallo8/logging-aggregation-system-d94f60f92dd0)

**References**:

- [https://docs.spring.io/spring-boot/docs/2.1.8.RELEASE/reference/html/howto-logging.html](https://docs.spring.io/spring-boot/docs/2.1.8.RELEASE/reference/html/howto-logging.html)
- [https://github.com/logfellow/logstash-logback-encoder#encoders--layouts](https://github.com/logfellow/logstash-logback-encoder#encoders--layouts)
- [https://stegard.net/2021/02/spring-boot-http-access-logging-in-three-steps/](https://stegard.net/2021/02/spring-boot-http-access-logging-in-three-steps/)
- [https://github.com/spring-projects/spring-boot/blob/v3.2.0/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/defaults.xml](https://github.com/spring-projects/spring-boot/blob/v3.2.0/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/defaults.xml)
- [https://sematext.com/glossary/structured-logging/#:~:text=Structured%20logging%20is%20the%20practice,application%20or%20an%20interested%20individual](https://sematext.com/glossary/structured-logging/#:~:text=Structured%20logging%20is%20the%20practice,application%20or%20an%20interested%20individual).
- [https://springframework.guru/using-logback-spring-boot/](https://springframework.guru/using-logback-spring-boot/)
- [https://logback.qos.ch/manual/configuration.html](https://logback.qos.ch/manual/configuration.html)
- [https://www.baeldung.com/spring-boot-embedded-tomcat-logs](https://www.baeldung.com/spring-boot-embedded-tomcat-logs)
- [https://github.com/akkinoc/logback-access-spring-boot-starter](https://github.com/akkinoc/logback-access-spring-boot-starter)
- [https://cassiomolin.com/programming/log-aggregation-with-spring-boot-elastic-stack-and-docker/](https://cassiomolin.com/programming/log-aggregation-with-spring-boot-elastic-stack-and-docker/)