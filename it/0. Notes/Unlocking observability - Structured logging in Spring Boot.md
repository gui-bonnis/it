## Well-defined structured logs can make Spring Boot applications more **observable** than free-form text logs

In this article, we will go through a step-by-step process to make Spring Boot application logs **queryable, contextual** and **analyzable**. Using a companion Java project, we will introduce you to the world of **structured logging and illustrate its benefits**.

![](https://miro.medium.com/v2/resize:fit:700/1*pQB6ydVnG5enPr4pyEh41w.jpeg)

_Source:_ [_https://geek-and-poke.com/geekandpoke/2015/10/18/why-logging-is-so-important_](https://geek-and-poke.com/geekandpoke/2015/10/18/why-logging-is-so-important)

In the end, we want **logs to be a data source for introspection of our application.** We want to be able to answer important questions ranging from “**what causes our application to return 400 status**” and “**how many of our requests get 400 statuses**” to contextual analyses like **“what led to this method being called”**.

# But first, what is structured logging?

Structured logging is a method of logging information from an application so that there is a defined structure to the payload instead of free-form text. This is typically done using JSON or XML.

The main advantages structured logging has over unstructured logging are: **machine readability, consistent format across applications, contextual information, integrating with other observability tools**. Structured logs greatly **improve a developer’s ability to introspect about a system** — be it to trace the lifetime of a request or to find a root cause of a bug.

# Companion project

To help guide you further, through this article, **we’ll demonstrate the use of structured logging with the help of a Github project called** [**Jamboree**](https://github.com/mourjo/jamboree), which allows its users to create social gatherings (or parties).

The project uses the [ELK stack for storing structured logs](https://www.elastic.co/elastic-stack) and is built using [Spring Boot](https://spring.io/projects/spring-boot/) on Java. All the screenshots in this blog post are taken here: [https://github.com/mourjo/jamboree/](https://github.com/mourjo/jamboree/)

# The key benefits of structured logging

The [Oxford Dictionary](https://www.oxfordlearnersdictionaries.com/definition/english/introspection) defines introspection as “**the careful examination of your own thoughts, feelings and reasons for behaving in a particular way**”. Similarly, a software application logs information to help us (its developers and maintainers) understand and examine its behavior and debug issues in production.

But, **it can be hard to understand or easily examine those logs that your application creates**. That’s where structured logging can really help. With the help of our companion project, we’ll show you some of the key benefits with structured logging.

## Enhanced queryability

Application logs in their infancy are produced as text and can **quickly become verbose.** Consider the image below — on the left side is a screenshot of a Java application emitting logs and on the right side is a logging UI (called Kibana) that allows searching logs by fields.

In the screenshot, we are interested in finding out when a certain entity was created. As you can see, **it is far easier to extract this information if structured logging is used**:

![](https://miro.medium.com/v2/resize:fit:700/1*GIWrDwKozBkmMwk46GBBNg.png)

Finding one piece of useful information from raw logs emitted by applications can be difficult without enhanced queryability through stuctured logging

Structured logging can help **uncover some behavioral traits** by introspecting the logs. Consider trying to answer why there are some 400 response codes observed by clients. With structured logging, we **can search logs specifically for this error code**. We clearly see that it is **only happening on the “POST /party/” route** and is due to missing parameters.

![](https://miro.medium.com/v2/resize:fit:700/1*0WaQXMuV7znqGvfT11JOvA.png)

Structured logging can help uncover behavioral traits — here the 400 error codes only happen on the POST /party/ route

## Standard fields in logs

We can show **rich contextual information other than just the log message** (eg, including the request ID). This ensures that fundamental domain-specific fields are always added to log statements even if the logged message does not have it. In the screenshot below, we see most logs have the HTTP URI and the method fields. Having these standard fields, **we could search for logs originating from a particular route**.

![](https://miro.medium.com/v2/resize:fit:700/1*LmIJvB-0t4uXZq_kBsqJEg.png)

Contextual information like URI and request method may not be present in the log message itself, but can be ingested by the application as extra data through structured logging

## Deeper insights through visualizations

Logs are a stream of information coming from our software application. Structured logging allows us to look at the macro picture through the **lens of aggregated information** — like the number of requests to an endpoint over time, or the number of 404s — aggregated information like this is shown in the image below.

![](https://miro.medium.com/v2/resize:fit:700/1*4Q66hAchHBUKEf1svCrLGQ.png)

Structured logging as source of data can be aggregated to view overall statistics like the percent of requests that end with a 4xx error

# How to add structured logging with SLF4J

In this section we describe how to add structured logs using [SLF4J](https://www.slf4j.org/) (or Simple Logging Facade for Java). There are three main components that we need to configure:

1. **Log format:** By default an application will produce logs in a text format — we have to add structure to these logs by defining a format. In this blog, we will use a **structured JSON format**.
2. **Log generation:** We need to **configure our application to include contextual information** in addition to the log message, like a user-id who made a request.
3. **Log ingestion:** Ultimately we have to ingest these logs into a **data store that allows us to interact with the logs**. We will use logstash to push application logs into our storage engine Elasticsearch and we will use Kibana as the logging UI.

## Step 1: Generating logs

In our [demo application](https://github.com/mourjo/jamboree), we choose to use [SLF4J](https://www.slf4j.org/) with [Logback](https://logback.qos.ch/index.html), which is a commonly used logging framework. This is defined in the [project dependencies](https://github.com/mourjo/jamboree/blob/main/pom.xml#L32-L42).

![](https://miro.medium.com/v2/resize:fit:700/1*uKGAY6oo6nxpKxcrLf7ecQ.jpeg)

SLF4J is a logging facade that works with many logging backends, in this article, we demonstrate structured logging with SLF4J and Logback as a backend

Adding logs is a two-step process:

1. Add a logger in a class — [example in the repository](https://github.com/mourjo/jamboree/blob/main/src/main/java/me/mourjo/jamboree/rest/PartyController.java#L25C1-L26C1)
2. Generate logs [using levels](https://www.slf4j.org/api/org/apache/log4j/Level.html) like INFO or ERROR — [see this line in the repository](https://github.com/mourjo/jamboree/blob/main/src/main/java/me/mourjo/jamboree/rest/PartyController.java#L36)

public class PartyController {  
    private final Logger logger = LoggerFactory.getLogger(getClass().getName());  
  
    @Operation  
    @GetMapping("/")  
    public Map<String, String> index() {  
        logger.info("Reading index path");  
        return Map.of("message", "Welcome to Jamboree!");  
    }  
}

This will generate logs in a text-only format. To add a structure to the logs, we have to use **MDC or Mapped Diagnostic Context**. MDC will add contextual information that will be present along with the log message. MDC is a map maintained by the logging framework ([Logback](https://github.com/qos-ch/logback/blob/master/logback-classic/src/main/java/ch/qos/logback/classic/util/LogbackMDCAdapter.java) in the demo project) where the application code provides **key-value pairs** which can then be inserted by the logging framework in log messages. This key-value pair way of adding logs is what adds structure to the log messages.

The MDC class contains static methods which can be used to add keys to any log message that is subsequently generated by the application:

MDC.put("REQUEST_URI", request.getRequestURI());  
   
if (params.location() == null) {  
logger.error("Missing parameter: location");  
       return Response.error(HttpStatus.BAD_REQUEST, "Location is mandatory.");  
}  
  
// clean up  
MDC.remove("REQUEST_URI");

With the contextual information being present at the time of log generation, the format of the log can use this to generate rich information that is useful when reading the logs.

## Step 2: Defining the log format

In the previous snippet, we added `REQUEST_ID` with MDC. Let us now configure the log format to introduce a more structured format that will generate log messages as a **JSON payload containing keys and values**. JSON formatting is required because many tools, including our storage engine, supports it out of the box. To do this, we have to configure logback settings.

## **Logback configuration**

Largely, logback defines three settings for defining the log format:

- **Logger**: This is the application class that will produce the log. This is defined in the [source code like this](https://github.com/mourjo/jamboree/blob/341e61a9e6ef9ec93672e6e95d328d9e69f6b387/src/main/java/me/mourjo/jamboree/service/PartyService.java#L18).
- **Appender**: The [appender](https://logback.qos.ch/manual/appenders.html) receives the logs produced by all the loggers in an application and stores them in the defined format — this could be a file, for instance. In our example project, we have two appenders — one to print the logs [to the console](https://github.com/mourjo/jamboree/blob/341e61a9e6ef9ec93672e6e95d328d9e69f6b387/src/main/resources/logback.xml#L4) and another “rolling appender” to [store logs in the JSON](https://github.com/mourjo/jamboree/blob/341e61a9e6ef9ec93672e6e95d328d9e69f6b387/src/main/resources/logback.xml#L17) format to a directory.
- **Layout**: The [layout](https://logback.qos.ch/manual/layouts.html) defines how a single log should be formatted. For our console appender, the output format contains the log message and the log level, while the rolling appender will save it a JSON payload.

Following is the logback configuration taken [from the demo repository](https://github.com/mourjo/jamboree/blob/341e61a9e6ef9ec93672e6e95d328d9e69f6b387/src/main/resources/logback.xml) and defines an appender called ROLLING, creates one file for all logs per minute using the JSON format. We also have the original console appender which makes logs more human readable.

<appender name="ROLLING" class="ch.qos.logback.core.rolling.RollingFileAppender">  
    <file>logs/jamboree.log</file>  
  
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
        <fileNamePattern>logs/jamboree-%d{yyyy-MM-dd-HH-mm}.log</fileNamePattern>  
        <maxHistory>60</maxHistory>  
        <totalSizeCap>20GB</totalSizeCap>  
    </rollingPolicy>  
  
    <layout class="ch.qos.logback.contrib.json.classic.JsonLayout">  
        <jsonFormatter class="ch.qos.logback.contrib.jackson.JacksonJsonFormatter">  
            <prettyPrint>false</prettyPrint>  
        </jsonFormatter>  
        <timestampFormat>yyyy-MM-dd HH:mm:ss.SSS</timestampFormat>  
        <appendLineSeparator>true</appendLineSeparator>  
    </layout>  
</appender>

With this configuration, the logs are going to be available in the logs directory as indicated by the file pattern:

$ head logs/jamboree-2023-12-04-08-48.log  
{"timestamp":"2023-12-04 08:48:00.106","level":"INFO","thread":"http-nio-7123-exec-2","mdc":{"REQUEST_URI":"/party/","TRACE_ID":"e10d91d9-b822-40cb-8794-45a103b6c248","REQUEST_ID":"66481f10-53ee-4c88-b9b2-8547b1416b85","PARTY_ID":"12442","REQUEST_METHOD":"POST"},"logger":"me.mourjo.jamboree.rest.PartyController","message":"Creating a party with PartyRequest[name=Adi Dhakeswari, location=Kolkata, time=null]","context":"default"}  
{"timestamp":"2023-12-04 08:48:00.113","level":"ERROR","thread":"http-nio-7123-exec-2","mdc":{"REQUEST_URI":"/party/","TRACE_ID":"e10d91d9-b822-40cb-8794-45a103b6c248","REQUEST_ID":"66481f10-53ee-4c88-b9b2-8547b1416b85","PARTY_ID":"12442","REQUEST_METHOD":"POST"},"logger":"me.mourjo.jamboree.rest.PartyController","message":"Missing parameter: time","context":"default"}

## Step 3: Ingestion into logging infrastructure

To make the most out of structured logs, we need to **index the JSON logs into a search engine that facilitates the complex search queries and analysis tools** we showed above.

To do this, our logging infrastructure will contain the following components as shown below — in Jamboree, all of this is configured to run through `[docker compose](https://github.com/mourjo/jamboree/blob/main/docker-compose.yml)`.

![](https://miro.medium.com/v2/resize:fit:700/1*r8idPzOxcTxWAVYHzyKC6A.png)

The logging infrastructure for our demo project uses the [ELK](https://www.elastic.co/elastic-stack) stack which runs inside [docker containers](https://github.com/mourjo/jamboree/blob/main/docker-compose.yml)

In the above image, there are three main components involved in the logging process:

- **Log storage**: This is the search index and storage location for our structured logs. For this demo, we are using [Elasticsearch](https://www.elastic.co/elasticsearch)
- **Log ingestor**: Application logs are locally stored on files (or streamed to STDERR), these logs need to be aggregated and collected for storage in Elasticsearch. In a production setup, there will be multiple nodes from where logs need to be aggregated. In this demo, we only have one application instance producing these nodes, which are periodically fetched by [Logstash](https://www.elastic.co/logstash).
- **Logging UI**: As logs get aggregated and stored, we need an user-interface to view and analyze the logs. We will use [Kibana](https://www.elastic.co/kibana) for this.

This is a very common setup (commonly called [ELK](https://www.elastic.co/elastic-stack) or Elastic-Logstash-Kibana).

# Contextual information: What led to this method being called?

MDC provides static methods which manage **keys and values that are emitted with every log message**. Under the hood, this is done by [storing the keys and values in a thread’s local context](https://github.com/qos-ch/logback/blob/master/logback-classic/src/main/java/ch/qos/logback/classic/util/LogbackMDCAdapter.java#L47).

**The MDC context is persisted across nested method calls.** A method that adds some keys to MDC and then calls another method, which then logs some information will also be able to log the parent method’s key-value pairs.

For example, in the sample project when the [DatetimeFormat](https://github.com/mourjo/jamboree/blob/main/src/main/java/me/mourjo/jamboree/datetime/DatetimeFormat.java#L58) class logs a warning, **the log will also contain the associated information of** `**TRACE-ID**` **and** `**REQUEST-METHOD**` **although the class is unaware of it.**

![](https://miro.medium.com/v2/resize:fit:700/1*tsTWufoZhhcr99bOt-BHgQ.png)

The method that logs a piece of information may not have the contextual information that makes structured logging so useful — for example, the DateFormat class is unaware of the request method or trace ID but the log emitted by the application will include this information

This allows **grouping of different logs by the context** and this makes our logs tell a story of what really happened in one single request’s lifetime:

![](https://miro.medium.com/v2/resize:fit:700/1*y5osQTw97MnBzXGmVzqs0Q.png)

Contextual information like TRACE_ID can allow grouping logs that were generated only while processing a single request — it shows a timeline of all the logs as it were emitted as if it was the only thing happening in the application

## Logging context in a multi-threaded environment

MDC operations are backed by thread local storage, but if threads are reused, **care should be taken to clear the logging context**, which can otherwise lead to bugs like this where the log message seems to have a different ID than the ID in the MDC column:

![](https://miro.medium.com/v2/resize:fit:700/1*6lblCIkRioXeRiYde0iQOg.png)

Uncleared logging context can lead to unexplained behavior — like a different ID in the context to the ID in the log message

The anomaly above is due to [a bug in clearing of the logging context](https://github.com/mourjo/jamboree/blob/main/src/main/java/me/mourjo/jamboree/rest/PartyController.java) in past requests. For best results, we should clear up logging contexts either in the method that adds the context, like this:

try {  
 MDC.put("PARTY_ID", getPartyId());  
       doSomeWork(); /// other logic  
} finally {  
 /// note MDC.clear() will remove all keys including the ones not added by this method  
 MDC.clear();  
}

Another option for cleaning up logging context specifically in web servers, is to have a request interceptor or filter, that clears logging context after each request processing ends like [in this example](https://github.com/mourjo/jamboree/blob/main/src/main/java/me/mourjo/jamboree/middleware/LoggingInterceptor.java).

_Note: MDC provides an auto-cleaning up mechanism via_ [_MDCCloseable_](https://www.slf4j.org/api/org/slf4j/MDC.MDCCloseable.html) _— but this cleanup may not perform correctly and it is proposed to be deprecated, see_ [_SLF4J-557_](https://jira.qos.ch/browse/SLF4J-557)_._

## Thread Pool Support

**Logging context is preserved in one thread’s context**, and is propagated to any thread that the thread creates. However, in most modern applications, thread creation will be delegated to [Executors](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executors.html), which are responsible for managing a pool of threads. In this case, the logging context does not get transferred.

**MDC provides helpful methods** [**getCopyOfContextMap**](https://www.slf4j.org/api/org/slf4j/MDC.html#getCopyOfContextMap--) **and** [**setContextMap**](https://www.slf4j.org/api/org/slf4j/MDC.html#setContextMap-java.util.Map-)**,** which can be used when transferring work from one thread to another. This [commit in our demo project](https://github.com/mourjo/jamboree/commit/c0e91a075a588841f0fccf35e4c434ac9f01770f) fixes the problem of lost context when using executors.

# Macro analysis of logs

Structured logs are a source of rich data. As such, the same logs that we have been using above can be analyzed with visualizations in Kibana. For example, **we can use a simple query to find how many distinct parties were created in the last 15 minutes**:

![](https://miro.medium.com/v2/resize:fit:700/1*Usj2wSAHqb9mTIajZV17NQ.png)

Data ingested from fields in structured logs can be aggregated to view statistics like the number of views in a given time window

Note that the data source is still application logs, so very accurate analysis of numbers should not be done with these visualizations — but they do provide a high level analysis for understanding how our application is doing.

# Conclusion

With its easy searchability, machine readability and its ability to deliver contextual information (to name just a few benefits), **structured logging can really help increase efficiency, especially in regard to system observability**.

Structured logging can **streamline the more arduous and mundane processes teams have to endure**, such as monitoring and debugging applications. With its use, structured logging can help your team ensure the ongoing performance and reliability of your applications. To sum up:

- Structured logging allows aggregation of logs in **a standard format**, with **contextual information** that enable us to fully understand logs like a story
- Structured logging can be set up with **SLF4J using Mapped Diagnostic Context in Java**
- The **ELK stack** (Elastic-Logstash-Kibana) enables us to ingest logs from an application into the logging UI called Kibana
- Logs can also be used to analyze **macroscopic behavior** of the application using Kibana Visualizations
- The code used in this blog and the infrastructure setup is available here: [https://github.com/mourjo/jamboree/](https://github.com/mourjo/jamboree/)