# 📥 Kafka Consumers Guide
---

## 📘 Key Topics Covered

### 1. Introduction to Kafka Consumers

* Kafka consumers subscribe to one or more topics and read streams of records.
* They can be part of **consumer groups** to divide processing across multiple instances.
* Each partition is assigned to only one consumer in a group at a time, ensuring scalability and fault tolerance.

---

### 2. Subscribing and Unsubscribing

* **subscribe(List<String> topics)**: Allows a consumer to dynamically subscribe to one or more topics.
* **unsubscribe()**: Stops receiving records from all previously subscribed topics.
* Useful for adjusting topic focus at runtime.

---

### 3. Assigning Specific Partitions

* **assign(Collection<TopicPartition> partitions)**: Directly assigns specific partitions to the consumer.
* Useful for static assignment and fine-grained control.
* Consumers using `assign()` do not participate in group rebalancing.

---

### 4. The Poll Loop

* **poll(Duration timeout)**: Continuously polls Kafka for new records.
* Typically used inside a loop for ongoing record processing.
* Must be called periodically to avoid consumer group timeouts.

---

### 5. Offset Management

* **enable.auto.commit** (default: true): Automatically commits offsets periodically.
* **Manual commit options**:

  * `commitSync()` — Synchronous offset commit.
  * `commitAsync()` — Asynchronous commit for better throughput.
* Offsets track how much of the topic has been read.

---

### 6. Rebalancing and Partition Assignment

* Rebalancing occurs when:

  * A consumer joins or leaves the group.
  * Topics are added or removed.
* Kafka automatically reassigns partitions among group members.
* Partition assignment strategies:

  * **Range**: Assigns sequential partitions.
  * **Round-Robin**: Distributes partitions evenly.

---

### 7. Implementing a Kafka Consumer (Java)

**Sample steps**:

1. Configure:

   ```java
   Properties props = new Properties();
   props.put(\"bootstrap.servers\", \"localhost:9092\");
   props.put(\"group.id\", \"order-consumer-group\");
   props.put(\"key.deserializer\", \"org.apache.kafka.common.serialization.StringDeserializer\");
   props.put(\"value.deserializer\", \"org.apache.kafka.common.serialization.StringDeserializer\");
   ```
2. Create consumer:

   ```java
   KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
   consumer.subscribe(Arrays.asList(\"orders\"));
   ```
3. Poll and process:

   ```java
   while (true) {
       ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
       for (ConsumerRecord<String, String> record : records) {
           System.out.printf(\"offset = %d, key = %s, value = %s%n\", record.offset(), record.key(), record.value());
       }
   }
   ```

---

### 8. Best Practices

#### ✅ Error Handling

* Wrap record processing logic in try-catch blocks.
* Use logging to trace processing failures.

#### ✅ Resource Management

* Always call `consumer.close()` in a `finally` block or shutdown hook.

#### ✅ Offset Commit Strategy

* Prefer manual offset commits when exact-once or at-least-once delivery is required.
* Use synchronous commit (`commitSync()`) to guarantee delivery acknowledgment.

#### ✅ Monitoring and Metrics

* Monitor consumer lag, processing time, and rebalance frequency.
* Use tools like Prometheus, Grafana, or Kafka’s built-in JMX metrics.

---

## 🛠 Configuration Parameters

| Parameter            | Description                                 | Example                  |
| -------------------- | ------------------------------------------- | ------------------------ |
| `bootstrap.servers`  | Kafka cluster address                       | `localhost:9092`         |
| `group.id`           | Consumer group name                         | `order-processing-group` |
| `enable.auto.commit` | Whether offsets are committed automatically | `true/false`             |
| `auto.offset.reset`  | What to do when no initial offset is found  | `latest` / `earliest`    |
| `max.poll.records`   | Max records returned in one poll            | `500`                    |
| `session.timeout.ms` | Timeout to detect consumer failures         | `10000`                  |

