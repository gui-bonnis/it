Logging and Messaging aren’t the most glamorous aspects of software engineering but are necessary. Getting logging right, out of the gate, for your project is essential. Specifically, what should be in a structured log message?

We’ve built a semi-opinionated [structured-messaging library](https://github.com/erichosick/structured-messaging) that enables you to create structured logs using Winston.

![](https://miro.medium.com/v2/resize:fit:700/1*zvlhtd4njLwj_rdBc5bAgA.jpeg)

Original logging dates back hundreds of years.

The post is not about:

- storing messages or collecting all messages generated from different compute instances to a single destination.
- logging libraries such as [Winston](https://github.com/winstonjs/winston), [Bunyan](https://github.com/trentm/node-bunyan), etc.
- defining an industry standard such as [RFC5424 Syslog](https://tools.ietf.org/html/rfc5424#section-6.1), etc.
- Processing messages using tools like

This post discusses why you should structure your logs and what kind of information you may want to provide for each log instance.

# Logging and Messaging

The structure of logging and a message can be the same. However, usage varies greatly. Please see our post on [Messaging and Logging: Implementation and Intent](https://medium.com/full-stack-architecture/messaging-and-logging-implementation-and-intent-1a8ffc7bfe96).

# Why Structured Format For Messages and Logging?

Logging information using an ad-hoc format makes it difficult for people to gain insight into what is happening in a computing environment: this is especially the case for data analysts who may need to use the logs for just such a purpose.

Providing a consistent message format enables:

- systems like Apache Spark to easily consume and analyze logs
- automation of routing messages in [Kafka](https://kafka.apache.org/)
- reformating log messages for easy human review (_JSON_ -> _YAML_)

Structured logging requires a data format. We’ll recommend using _JSON_ in this article.

# Design Considerations

After reviewing different logging formats and chatting with people of different backgrounds and needs, here are some of our takeaways:

- Local Development: Logging can’t be too noisy. So, minimizing the number of elements in the message is needed.
- Local Development: Having an excellent visual representation of the log messages: red for errors, green for info, yellow for warnings, etc.
- Idempotence: Sending the same message twice doesn’t result in double counting.
- Verification: In some situations, we may need to verify that the message has not changed during transit.

# A Structured Message Format

Covered in detail later, the following is a complete message:

{  
  "id": "d96b195b-b24b-4981-8d28-46a68f07ff0a",  
  "level": "info",  
  "message": "Running SQL for Alan: 'SELECT * FROM types;'.",  
  "topics": ["log", "notification"],  
  "priority": "1",  
  "template": "Running SQL for %{data.name}: '%{data.query}'.",  
  "transactId": "63bd5a30-398a-4ebb-9e83-ec2c4e92aa89",  
  "sessionId": "5565d96b-2b1f-4889-8e66-5b994036d1bf",  
  "traceId": "e95d9220-62e1-42e5-a0c5-b158dab7f2ec",  
  "account": {  
    "memberId": "someuser",  
    "workspaceId": "some workspace",  
    ...  
  },  
  "time": {  
    "format": "iso8601",  
    "created": "2020-10-20T04:39:34+07:00",  
    "expires": "2020-12-20T09:00:34+07:00",  
  },  
  "pii": ["data.name"],  
  "dataSchema" : {  
    "name": "sql",  
    "ver": "1.0.0"  
  },  
  "data" : {  
    "query": "SELECT * FROM types;",  
    "name": "Alan",  
  },  
  "context" : {  
    "app": {  
      "env": "dev",  
      "name": "loginService",  
      "platform": "magicCrm",  
      "file": "/Users/me/proj/login-service/query.ts",  
      "line": 34,  
    },  
    "compute": {  
      "host": "localhost",  
      "shardId" : "9bc423dc-61f2-419c-9f53-8f417a74863e",  
      "processId": "4545",  
    },  
    "hostedBy": {  
      "name": "aws",  
      "___": "hosted by specific properties",  
    },  
  },  
  "hash": {  
    "algo": "Sha1",  
    "code": "4d90a79012905e89ea4473258e1874fcce12a593"  
  }  
}

## Starting With the Minimum

Running an application Locally would require a different message format from running an application in a company with hundreds of applications located within multiple cloud providers. IT would need to know which cloud provider the log message had originated. A developer running an application Locally isn’t necessarily interested in that information.

**However!** We want our message format to be consistent and relevant to whoever is consuming that message. So, we’ll start with a small message format and build on that.

The following JSON displays part of a message: a developer may want to use it when debugging locally.

{  
  "id": "d96b195b-b24b-4981-8d28-46a68f07ff0a",  
  "level": "info",  
  "message": "Running SQL for Alan: 'SELECT * FROM types;'."  
}

NOTE: The message above isn’t showing all the required fields, but the original message had all the required fields.

# Structured Message Properties

## id (required)

Every message generated is given a globally unique id. A system consuming a message can ensure it only stores the message once (should the same message somehow be sent twice). Systems designed in this manner are idempotent.

_Example_: `"id": “c7963876–135c-11eb-adc1–0242ac120002”`

NOTE: No specific [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier) type is recommended.

## level (required)

A log level enables a log message to be filtered based on the message's importance.

_Example_: `"level": "warn"` , `"level": "info"`

NOTE: **Different logging platforms use different log level enumerations**: a few examples being [Winston](https://github.com/winstonjs/winston#logging-levels), [Log4J](https://www.javatpoint.com/log4j-logging-levels), [log4net](http://logging.apache.org/log4net/release/manual/introduction.html), and python’s [logging](https://docs.python.org/3/library/logging.html#levels). It is essential to consider the differences because a large company may be using one or more of these logging libraries. Group like IT, infrastructure, dev-ops, and data science are negatively affected by different log levels.

## topics (required)

A topic helps in routing messages and is especially useful if you use services that route based on Topics: [Kafka](https://kafka.apache.org/), [Kinesis](https://aws.amazon.com/kinesis/), etc.

At the minimum, a message should contain a `log` or `event` topic.

A Topic can further scope the message's purpose: especially if it is an event.

_Examples_: `"topics": ["event", "auth", "login"]`, `"topics": ["event", "auth", "logout"]`

It may even be easier to consume messages on the backend if you add the log level in the topics: `"topics": ["log", "warn"],`, `"topics": ["log": "error"]`.

NOTE: We considered moving `level` to the `topic` property, but it is common practice to have a`level` property at the root of a log.

## priority (required)

Priority can further help with routing: enabling events to “jump” the processing queue. Different systems may have values that represent other priorities. Examples are Microsoft’s [message priority](https://docs.microsoft.com/en-us/dotnet/api/system.messaging.messagepriority?view=netframework-4.8), [Mail Kit](http://www.mimekit.net/docs/html/T_MimeKit_MessagePriority.htm), etc.

_Examples_: `“priority": 1`, `“priority": 2`

## message (required)

A simple human-readable message. The message is the final formatted message requiring no further processing.

_Examples_: `“message": "User logged out.”`, `“message": "Error opening file.”`

## template (optional)

The template field enables localization for error messages.

An original template such as:

`“template”: “Running SQL for ${data.name}: ‘${data.query}’.”`

could then be run through a translator resulting in :

`“template”: “Ejecutando SQL para ${data.name}: ‘${data.query}’.”`.

In these examples, we’re using Javascript’s [template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals). The expression contains a string that defines the location in the message where the value is pull from. For example, the expression `data.name` pulls the name value from the data object of the message. The expression `context.app.env` will pull the `env` value from the `app object`which is under the `context` object.

Further, pulling out the values that change for each message enables the message's reformatting based on the consumer. A developer's error message could be logged and then re-formatted to forward to the user of a front-end application.

## transactionId (required)

The `transactId` is used for transactions that span multiple services enabling a developer to follow the transaction to completion. The `tarnsactId` is generated at the transaction source: say, a button clicked by a user in the UI.

The `transactId` can be used for debugging: gaining insight into the performance aspects of your system.

NOTE: If a `transactId` has not been provided, the logging service would generate one.

NOTE: A `transactId`does require extra effort on the part of developers. For example, a developer would need to pass a `transactId` to the client's service via an HTTP header (for example) and pass the id on to other services.

_Examples_: `"transactId": "d5d98834–443d-4ac4-b3e0–5935fe476b56"`

## sessionId (optional)

The [sessionId](https://en.wikipedia.org/wiki/Session_ID) is a unique id used to manage a [session](https://en.wikipedia.org/wiki/Session_(computer_science)).

_Examples_: `"sessionId": "d9af77b4–1dfe-4a73-a8c2-f0c454e1dc70"`

## time (required)

The time object contains:

- **format (optional)** —the date format. We’re using [iso8601](https://en.wikipedia.org/wiki/ISO_8601) with the UTC offset.
- **created (required)** — the creation date and time of the message.
- **expires (optional)** —The expiry date and time of the message.

_Example_:

"time": {  
  "format": "iso8601",  
  "created": "2020-10-20T04:39:34+0700",  
  "expires": "2020-12-20T09:00:34+0700"  
}

## pii (optional)

There may be cases where information within the message is considered [personally identifiable information](https://en.wikipedia.org/wiki/Personal_data). Sending PPI in a message may be necessary to process the message. Try not to log PPI.

Providing a PII list enables a system to mangle the information based on who is viewing the data.

The PII property contains a string that defines which data will be obfuscated. The expression `data.email` would obfuscate the email address.

_Examples_: `"pii": ["data.email", "message"]` would result in the following output:

{  
  level: "warning",  
  message: "***************************",  
  data: {  
    "email": "s******r@*******.com"  
  }  
}

## schema (optional)

The schema object defines the name and version of the schema that could be applied to the data property when dehydrating the data: turning data into a native object instance. A schema enables tools like [J](https://github.com/sideway/joi)OI to validate the information located in the data property.

- **name (optional)** — the name or type of schema located in the data property.
- **ver (required)** — the schema version found in the data property.

_Example_:

"schema" : {  
  "name": "sql",  
  "ver": "1.0.0"  
}

## data (optional)

The data property provides an area where you can put data of any type. For example, you could put any information that varies in the message property in the data and take advantage of the template property.

_Example_:

"data" : {  
  "query": "SELECT * FROM types;",  
  "name": "Alan",  
}

**context (required)**

Context provides a picture of the situation (context) at the message creation time. Context is helpful when tracking down bugs.

- **app.env (required)**— the software’s environment. Examples: dev, stage, prod, etc.
- **app.name (optional)** — the name of the application/service.
- **app.platform (optional)** — the platform the application/service resides. A Platform is an aggregation of applications/services installed together.
- **app.file (optional)** — the file line number of the message.
- **compute.host (optional)** — the hostname of the application.
- **shardId (optional)** — if the application runs in a cluster, then the shardId is used to identify a compute instance within the cluster.
- **processId (optional)** — the process's id.
- **hostedBy.name (optional)** — the service or cloud platform name.
- **hostedBy.___(optional)** — properties specific to the host. For example, with AWS, you might include the region, the role the application was running under, etc.

_Example_:

"context" : {  
    "app": {  
      "env": "dev",  
      "name": "loginService",  
      "platform": "magicCrm",  
      "file": "/Users/me/proj/login-service/query.ts",  
      "line": 34,  
    },  
    "compute": {  
      "host": "localhost",  
      "shardId" : "9bc423dc-61f2-419c-9f53-8f417a74863e",  
      "processId": "4545",  
    },  
    "hostedBy": {  
      "name": "aws",  
      "___": "hosted by specific properties",  
    },  
  },

## hash (optional)

The hash code verifies message integrity. **NOTE!** Calculate the hash **before** adding the hash property to the message.

- **algo (required)**— the algorithm used to generate the hash value. Some example algorithms are Md5, Sha1, Sha256, Sha384, Sha512, Crc32, Crc32b, Gost, Whirlpool, Ripemd160, and Crypt hash.
- **code (required)** — the hash code value.

_Example_:

"hash": {  
  "algo": "Sha1",  
  "code": "4d90a79012905e89ea4473258e1874fcce12a593"  
}

NOTE: Property order also matters. As such, deserializing and then serializing the message may change the order of properties invalidating the hash code.

# Conclusion

Hopefully, this post has given you some ideas about what information you would like to include in your logs.