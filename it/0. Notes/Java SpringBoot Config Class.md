How can I create a configuration class in Java Springboot to load properties from application.properties files using @Data @Configuration annotations and be consumed by another class

###### ChatGPT said:

ChatGPT

To create a configuration class in Java Spring Boot that loads properties from the `application.properties` file and can be consumed by another class, you can follow these steps:

1. Create a `Configuration` class with `@Configuration` and `@Data` annotations. This will make the class a Spring configuration bean and generate getters and setters for the properties.

java

Copy code

`import org.springframework.beans.factory.annotation.Value; import org.springframework.context.annotation.Configuration; import lombok.Data;  @Configuration @Data public class MyAppConfig {      @Value("${myapp.property1}")     private String property1;      @Value("${myapp.property2}")     private int property2;      // Add more properties as needed      // Constructor, getters, and setters }`

2. In the above code, `@Value` annotations are used to inject the property values from the `application.properties` file. The `${myapp.property1}` and `${myapp.property2}` placeholders correspond to the property keys defined in the `application.properties` file.
    
3. Create an `application.properties` file in the `src/main/resources` directory of your project. Add your properties with their corresponding values.
    

Copy code

`myapp.property1=Hello World myapp.property2=42`

4. To consume the properties from the `MyAppConfig` class in another class, you can simply inject an instance of `MyAppConfig` using dependency injection.

java

Copy code

`import org.springframework.beans.factory.annotation.Autowired; import org.springframework.stereotype.Component;  @Component public class MyService {      private final MyAppConfig appConfig;      @Autowired     public MyService(MyAppConfig appConfig) {         this.appConfig = appConfig;     }      public void doSomething() {         System.out.println("Property 1: " + appConfig.getProperty1());         System.out.println("Property 2: " + appConfig.getProperty2());         // Access other properties as needed     } }`

In the above example, `MyService` class has a constructor that accepts an instance of `MyAppConfig` bean. Spring will automatically inject the `MyAppConfig` bean when creating an instance of `MyService`. You can then access the properties using the `appConfig` instance.

Remember to ensure that your Spring Boot application has the necessary dependencies, such as `spring-boot-starter`, in your build configuration to enable Spring Boot features and property loading.

That's it! You have created a configuration class to load properties from `application.properties` and consumed them in another class using dependency injection.

3.5

##### You said:

I would like to have another annotation in my configuration class, what would be @ConfigurationProperties(prefix = "XXXXXXXXXX"). Beside that the properties that I will use has a deeper level. Like queues.transaction.mq = QUEUE.DEV.Q1

###### ChatGPT said:

ChatGPT

If you want to use the `@ConfigurationProperties` annotation with a prefix and handle properties with a deeper level, you can modify your configuration class as follows:

java

Copy code

`import org.springframework.boot.context.properties.ConfigurationProperties; import org.springframework.context.annotation.Configuration; import lombok.Data;  @Configuration @ConfigurationProperties(prefix = "queues.transaction") @Data public class MyAppConfig {      private String mq;      // Add more properties as needed      // Constructor, getters, and setters }`

In this example, the `@ConfigurationProperties` annotation is used with the `prefix` attribute set to `"queues.transaction"`. This prefix specifies the common prefix for the properties that will be mapped to the fields of the configuration class.

Next, update your `application.properties` file to include the property with a deeper level:

Copy code

`queues.transaction.mq=QUEUE.DEV.Q1`

With this setup, the property `queues.transaction.mq` will be mapped to the `mq` field in the `MyAppConfig` class.

To consume the properties from the `MyAppConfig` class in another class, you can follow the same approach mentioned in the previous response:

java

Copy code

`import org.springframework.beans.factory.annotation.Autowired; import org.springframework.stereotype.Component;  @Component public class MyService {      private final MyAppConfig appConfig;      @Autowired     public MyService(MyAppConfig appConfig) {         this.appConfig = appConfig;     }      public void doSomething() {         System.out.println("MQ: " + appConfig.getMq());         // Access other properties as needed     } }`

In this example, the `mq` property from `MyAppConfig` can be accessed using `appConfig.getMq()`.

Remember to update your project's dependencies to include the necessary Spring Boot libraries, such as `spring-boot-configuration-processor`, to enable the processing of `@ConfigurationProperties` annotations.

With these changes, you can use the `@ConfigurationProperties` annotation with a prefix and handle properties with a deeper level in your configuration class.

3.5

##### You said:

OK, good. It is possible to have a complex object at MyAppConfig class instead of String mq. For example: private MySubClass transaction; And MySubClass has the property private String mq;

###### ChatGPT said:

ChatGPT

Yes, it is possible to have a complex object in the `MyAppConfig` class, such as `private MySubClass transaction;`, where `MySubClass` has its own properties. Here's an example of how you can achieve that:

1. Define the `MySubClass` with its properties:

java

Copy code

`public class MySubClass {      private String mq;      // Constructor, getters, and setters }`

2. Modify the `MyAppConfig` class to use the `MySubClass` object:

java

Copy code

`import org.springframework.boot.context.properties.ConfigurationProperties; import org.springframework.context.annotation.Configuration; import lombok.Data;  @Configuration @ConfigurationProperties(prefix = "queues.transaction") @Data public class MyAppConfig {      private MySubClass transaction;      // Constructor, getters, and setters }`

3. Update the `application.properties` file to include the property with the complex object structure:

Copy code

`queues.transaction.transaction.mq=QUEUE.DEV.Q1`

In this example, the property `queues.transaction.transaction.mq` is mapped to the `mq` field inside the `transaction` object of the `MyAppConfig` class.

4. To consume the properties from the `MyAppConfig` class, you can use the same approach mentioned earlier:

java

Copy code

`import org.springframework.beans.factory.annotation.Autowired; import org.springframework.stereotype.Component;  @Component public class MyService {      private final MyAppConfig appConfig;      @Autowired     public MyService(MyAppConfig appConfig) {         this.appConfig = appConfig;     }      public void doSomething() {         System.out.println("MQ: " + appConfig.getTransaction().getMq());         // Access other properties of MySubClass and MyAppConfig as needed     } }`

In the `MyService` class, you can access the `mq` property using `appConfig.getTransaction().getMq()`.

With these modifications, you can have a complex object, `MySubClass`, as a property in the `MyAppConfig` class, and access its properties in the consuming class.