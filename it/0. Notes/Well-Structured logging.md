> It is widely known that the cost of the software for the most part does not consist of the costs of the initial development, but rather the health maintenance, making changes, and system development

Every application that works in production may fail. And, as engineers, we should investigate the reasons for those failures. Logs very often are very helpful in understanding what happened, in which order, and with which data. However, to be helpful, logs should be appropriately written and structured.

At the beginning let’s look at examples of bad logging.

# Bad logs examples

![](https://miro.medium.com/v2/resize:fit:700/1*Uos-jlIyn99M7JxhaPaP2w.png)

Picture 1 — Stuck logs

In picture 1 we can see that logs are stuck. When logs are stuck, we can’t create histograms, we can’t count them and it can really be an obstacle to understand when the problem began and how many users were affected. Therefore, avoid sticking of logs.

![](https://miro.medium.com/v2/resize:fit:700/1*xDxoRZulE78rC5UolNCMzQ.png)

Picture 2 — Unstructured logs

Is it easy to find the status of a response in picture 2? I don’t think so. Here we can see that the default nginx logs format is written into logs as-is. Well, you can do this, you can even filter logs, but reading it is not very convenient.

![](https://miro.medium.com/v2/resize:fit:700/1*kDDGAuLK1fwSkrqAPmZKhQ.png)

Picture 3 — Useless logs

Which message was processed? What we can know from this log? It seems that there is no big difference whether we have logs or not with logs of such quality.

# Logs format

Okay, now let’s look at how looks like good logs. But before we need to define what types of logs we may have.

1. Errors/Exceptions — **Error logs**  
    — Infrastructure (database, network, …)  
    — Unexpected Error (get [0] of null, …)
2. Expected Exceptions — **Warning logs**  
    — Validation Errors  
    — Business rules (order can’t be moved to state “In Progress” from state “Completed”)
3. Information — **Info logs**  
    — order successfully updated  
    — record marked as removed

Each log record must contain a “type” attribute to allow you to filter logs and create diagrams and histograms by different log types.

Writing error logs principles:  
1. Log’s message should have business meaning  
2. Log’s message should be unique in the codebase  
2. Log’s body should contain useful debug info  
3. Log’s body should contain an error message  
4. Log’s body should contain an error stack trace

// pay attention: error message doesn't contain variables  
// as a result, we will need it to not only filter log records,  
// but also to group them  
logger.error('Agreement update error', {  
  // we always put errors in error field  
  error,  
  // we add additional information to get know more  
  // about exception cicrumstances  
  agreementId,  
  userId,  
  status,  
});  
  
// log message:  
{  
  "type": "error",   
  "message": "Agreement update error",  
  "agreementId": "324233",  
  "userId": "23423",  
  "status": "ready"  
  "error": {  
    "message": "Forbidden operation, related document has status pending",  
    "stacktrace": "..."  
  },  
  ...  
}

Writing warning logs  
1. Log’s message should have business meaning  
2. Log’s message should be unique in the codebase  
2. Log’s body should contain useful debug info  
3. Log’s body should contain information about cause

logger.warn('Agreement validation exception', {  
  agreement,  
  error,  
  userId,  
});  
  
// log message:  
{   
  "type": "warn",  
  "message": "Agreement validation exception",  
  "agreement": {"id": 354, ...},  
  "error": {"reason": "Invalid status: null"},  
  "userId": "23423",  
  ...  
}

Info logs  
1. Log’s message should have business meaning  
2. Log’s body should contain useful debug info

logger.info('Agreement status is updated', {  
  agreementId,  
  userId,  
  status,  
});  
  
// log message:  
{   
  "type": "info",  
  "message": "Agreement status is updated",  
  "agreementId": "324233",  
  "userId": "23423",  
  "status": "ready",  
  ...  
}

Each log record must contain additional attributes, that are common for all logs. These attributes can be different and may depend on your environment. These additional attributes can be added to your logs by infrastructure services. I will provide just a few examples of such attributes:  
— pid  
— hostname  
— environment  
— POD-name  
— cluster-name  
— timestamp  
— …

As a result, your logs may look like:

// log message:  
{   
  "pid": 2341,  
  "env": "production",  
  "pod": "main",  
  "timestamp": "2023-09-27T21:23:51.352Z",  
  ...  
  "data": {   
    "type": "info",  
    "message": "Agreement status is updated",  
    "agreementId": "324233",  
    "userId": "23423",  
    "status": "ready",  
    ...  
  }  
}

Nowadays there are many tools for collecting, transferring, and storing logs. All these tools may have their specific requirements for logs. For example, Elasticsearch has the following constraints for JSON logs:  
— consistent data types (avoid putting different data types by the same paths. You can convert all non-object values to strings)  
— no dots in keys  
— limit for depth (after some depth, you should serialize objects to strings to avoid too deep nested objects)  
Therefore, for writing logs according to your logs storage, you, probably, need to clean and prepare them.

_Also, if you for some reason want to serialize logs’ data manually, don’t forget that objects, that you may want to put into logs, may have circular references._

# Example of well-structured logs

![](https://miro.medium.com/v2/resize:fit:700/1*mCaqP45Fh9wqZw8YWq_l8w.png)

Picture 4 — structured logs

In picture 4 we can see that now we can separate attributes in different columns, show and hide those columns. It allows us to avoid unnecessary information noise and helps us faster and easier find exceptions’ reasons.

![](https://miro.medium.com/v2/resize:fit:700/1*ng5OtavdpMaLPF4D_6eC3w.png)

Picture 5 — Logs histograms

Histograms are not something unique. You could build it even if your message attribute contains variables. But there is something that you can’t do if the logs’ message attribute contains variable values.

![](https://miro.medium.com/v2/resize:fit:700/1*Fuam4NrfXKHlJAGvRvhOCg.png)

Picture 6 — Error logs pie chart

With pie charts, you can easily understand which errors occur more frequently and require more attention. It can be very helpful when your system starts to behave strangely.

Putting log messages and accompanying data into separate attributes allows us to work with logs in a very convenient way. But, at the same time, it requires discipline. You can create special conventions, or tooling to add prefixes to logs. It depends on the tooling that you use and the ecosystem of your programming language.

# In conclusion

Every system requires specific logs treatment. Some systems have hundreds of thousands of users and they may produce billions of log records. In such systems, you can’t write every action to info logs. You need to add logs carefully and write only necessary log records. In some cases, you need to avoid writing into logs personal data or banking card numbers. Sometimes you need more than 3 types of logs and you can define your own log types. This post is prepared for demonstration of main ideas and it should show you how you can think about logs and how to make them more useful.