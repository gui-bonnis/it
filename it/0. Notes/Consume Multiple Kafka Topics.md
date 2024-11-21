How can I consume 3 different kafka topics in the same microservice

###### ChatGPT said:

ChatGPT

To consume multiple Kafka topics within the same microservice, especially in a Java Spring Boot context, you can follow a structured approach using **Spring Kafka**. Here are the key steps:

### Steps to Consume Multiple Topics

#### 1. **Add Kafka Dependencies**

Make sure your `pom.xml` (for Maven) or `build.gradle` (for Gradle) includes the Spring Kafka dependencies.

**Maven:**

xml

Copy code

`<dependency>     <groupId>org.springframework.kafka</groupId>     <artifactId>spring-kafka</artifactId>     <version>3.0.0</version> <!-- Adjust version as needed --> </dependency>`

**Gradle:**

groovy

Copy code

`implementation 'org.springframework.kafka:spring-kafka:3.0.0'`

---

#### 2. **Kafka Configuration**

Create a Kafka consumer configuration class to define how the consumers should behave.

java

Copy code

`import org.apache.kafka.clients.consumer.ConsumerConfig; import org.apache.kafka.common.serialization.StringDeserializer; import org.springframework.context.annotation.Bean; import org.springframework.context.annotation.Configuration; import org.springframework.kafka.annotation.EnableKafka; import org.springframework.kafka.core.ConsumerFactory; import org.springframework.kafka.core.DefaultKafkaConsumerFactory; import org.springframework.kafka.listener.ConcurrentMessageListenerContainer; import org.springframework.kafka.listener.config.ContainerProperties; import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;  import java.util.HashMap; import java.util.Map;  @EnableKafka @Configuration public class KafkaConsumerConfig {      @Bean     public ConsumerFactory<String, String> consumerFactory() {         Map<String, Object> props = new HashMap<>();         props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");         props.put(ConsumerConfig.GROUP_ID_CONFIG, "my-group");         props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);         props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);         return new DefaultKafkaConsumerFactory<>(props);     }      @Bean     public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {         ConcurrentKafkaListenerContainerFactory<String, String> factory =             new ConcurrentKafkaListenerContainerFactory<>();         factory.setConsumerFactory(consumerFactory());         factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL);  // Optional for manual commit         return factory;     } }`

---

#### 3. **Kafka Listener for Multiple Topics**

Define a listener method that listens to multiple Kafka topics. You can either process all topics in one method or have separate listener methods.

##### **Single Listener for Multiple Topics:**

java

Copy code

`import org.springframework.kafka.annotation.KafkaListener; import org.springframework.stereotype.Service;  @Service public class KafkaConsumerService {      @KafkaListener(topics = {"topic1", "topic2", "topic3"}, groupId = "my-group")     public void consume(String message) {         System.out.println("Received message: " + message);     } }`

##### **Separate Listeners for Each Topic:**

java

Copy code

`@Service public class KafkaConsumerService {      @KafkaListener(topics = "topic1", groupId = "my-group")     public void consumeTopic1(String message) {         System.out.println("Received from topic1: " + message);     }      @KafkaListener(topics = "topic2", groupId = "my-group")     public void consumeTopic2(String message) {         System.out.println("Received from topic2: " + message);     }      @KafkaListener(topics = "topic3", groupId = "my-group")     public void consumeTopic3(String message) {         System.out.println("Received from topic3: " + message);     } }`

---

#### 4. **Handling Concurrent Consumers**

You can adjust the concurrency level for each topic if needed by setting the `concurrency` property in the listener factory.

java

Copy code

`@Bean public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {     ConcurrentKafkaListenerContainerFactory<String, String> factory =         new ConcurrentKafkaListenerContainerFactory<>();     factory.setConsumerFactory(consumerFactory());     factory.setConcurrency(3);  // Adjust concurrency based on your needs     return factory; }`

---

#### 5. **Manual Offset Management (Optional)**

If you need fine-grained control over message acknowledgment, set the `AckMode` to manual and use the `Acknowledgment` parameter in your listeners.

java

Copy code

`@KafkaListener(topics = "topic1", groupId = "my-group") public void consumeTopic1(String message, Acknowledgment ack) {     System.out.println("Received from topic1: " + message);     ack.acknowledge();  // Manually commit the offset }`

---

### Summary

With this setup:

- You can either use a **single listener** to consume multiple topics or **separate listeners** for each topic.
- Adjust **concurrency** to manage performance.
- Use **manual acknowledgment** if needed for finer control over offset management.

This approach ensures that all your Kafka topics are consumed efficiently within the same microservice.