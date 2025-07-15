[返回](/mongodb/technolog/index)


# 使用 MongoDB Kafka Connector 与 Spring Boot 实现 CDC

下面是一个完整的方案，展示如何使用 MongoDB Kafka Connector 将变更事件发送到 Kafka，并在 Spring Boot 应用中消费这些变更事件。

## 1. 整体架构

```
MongoDB → MongoDB Kafka Connector → Kafka → Spring Boot Consumer → 业务处理
```

## 2. 环境准备

### 所需组件
- MongoDB 3.6+ (建议4.0+)
- Kafka 2.5+
- MongoDB Kafka Connector 1.7+
- Spring Boot 2.7+

## 3. 配置 MongoDB Kafka Connector

### 3.1 下载连接器
```bash
wget https://repo1.maven.org/maven2/org/mongodb/kafka/mongo-kafka-connect/1.7.0/mongo-kafka-connect-1.7.0-all.jar
mv mongo-kafka-connect-1.7.0-all.jar /path/to/kafka/connect/plugins/
```

### 3.2 创建连接器配置文件 `mongo-source.properties`

```properties
name=mongo-source-connector
connector.class=com.mongodb.kafka.connect.MongoSourceConnector
tasks.max=1
topics=mongo.changes

# MongoDB 连接配置
connection.uri=mongodb://username:password@localhost:27017

# 要监控的数据库和集合
database=inventory
collection=products

# 捕获的变更类型(insert,update,replace,delete)
pipeline=[{"$match": {"operationType": {"$in": ["insert", "update", "replace", "delete"]}}}]

# 输出格式配置
output.format.value=json
output.json.formatter=com.mongodb.kafka.connect.source.json.formatter.SimplifiedJson
```

### 3.3 启动连接器
```bash
bin/connect-standalone.sh config/connect-standalone.properties mongo-source.properties
```

## 4. Spring Boot 消费者实现

### 4.1 添加 Maven 依赖
```xml
<dependencies>
    <!-- Spring Kafka -->
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
    </dependency>
    
    <!-- MongoDB BSON 解析 -->
    <dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>bson</artifactId>
        <version>4.7.1</version>
    </dependency>
    
    <!-- JSON 处理 -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>
</dependencies>
```

### 4.2 配置 application.yml
```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      group-id: mongo-cdc-group
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
    listener:
      ack-mode: MANUAL_IMMEDIATE
```

### 4.3 创建消费者服务

```java
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.support.Acknowledgment;
import org.springframework.stereotype.Service;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

@Service
public class MongoChangeConsumer {
    
    private static final Logger log = LoggerFactory.getLogger(MongoChangeConsumer.class);
    
    private final ObjectMapper objectMapper = new ObjectMapper();

    @KafkaListener(topics = "mongo.changes")
    public void consume(String message, Acknowledgment ack) {
        try {
            JsonNode changeEvent = objectMapper.readTree(message);
            
            String operationType = changeEvent.get("operationType").asText();
            String documentId = changeEvent.get("documentKey").get("_id").get("$oid").asText();
            
            log.info("Received {} event for document ID: {}", operationType, documentId);
            
            switch (operationType) {
                case "insert":
                    handleInsert(changeEvent.get("fullDocument"));
                    break;
                case "update":
                    handleUpdate(documentId, changeEvent.get("updateDescription"));
                    break;
                case "delete":
                    handleDelete(documentId);
                    break;
                default:
                    log.warn("Unhandled operation type: {}", operationType);
            }
            
            ack.acknowledge();
        } catch (Exception e) {
            log.error("Error processing change event: {}", message, e);
            // 可根据业务需求决定是否重试
        }
    }
    
    private void handleInsert(JsonNode document) {
        // 处理新增文档逻辑
        log.info("New document inserted: {}", document);
        // 示例: 同步到其他系统或更新缓存
    }
    
    private void handleUpdate(String documentId, JsonNode updateDescription) {
        // 处理更新文档逻辑
        log.info("Document {} updated. Updated fields: {}", 
            documentId, updateDescription.get("updatedFields"));
        // 示例: 只更新变更的字段到缓存
    }
    
    private void handleDelete(String documentId) {
        // 处理删除文档逻辑
        log.info("Document {} deleted", documentId);
        // 示例: 从缓存中移除
    }
}
```

### 4.4 配置反序列化器（可选）

如果需要更复杂的反序列化逻辑，可以创建自定义反序列化器：

```java
public class MongoChangeDeserializer implements Deserializer<MongoChangeEvent> {
    
    private final ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public MongoChangeEvent deserialize(String topic, byte[] data) {
        try {
            return objectMapper.readValue(data, MongoChangeEvent.class);
        } catch (IOException e) {
            throw new SerializationException("Error deserializing Mongo change event", e);
        }
    }
}

// 对应的POJO
public class MongoChangeEvent {
    private String operationType;
    private DocumentKey documentKey;
    private JsonNode fullDocument;
    private UpdateDescription updateDescription;
    
    // getters and setters
    // 内部静态类 DocumentKey, UpdateDescription 等
}
```

## 5. 变更事件数据结构示例

### 插入事件
```json
{
  "_id": {
    "_data": "82607..."
  },
  "operationType": "insert",
  "clusterTime": {
    "$timestamp": {
      "t": 1620000000,
      "i": 1
    }
  },
  "fullDocument": {
    "_id": {
      "$oid": "5f8d7f3e6c5e6d4a9c8b9a7b"
    },
    "name": "智能手机",
    "price": 599.99,
    "stock": 100
  },
  "ns": {
    "db": "inventory",
    "coll": "products"
  },
  "documentKey": {
    "_id": {
      "$oid": "5f8d7f3e6c5e6d4a9c8b9a7b"
    }
  }
}
```

### 更新事件
```json
{
  "_id": {
    "_data": "82608..."
  },
  "operationType": "update",
  "clusterTime": {
    "$timestamp": {
      "t": 1620000001,
      "i": 1
    }
  },
  "ns": {
    "db": "inventory",
    "coll": "products"
  },
  "documentKey": {
    "_id": {
      "$oid": "5f8d7f3e6c5e6d4a9c8b9a7b"
    }
  },
  "updateDescription": {
    "updatedFields": {
      "stock": 95
    },
    "removedFields": []
  }
}
```

## 6. 高级配置与优化

### 6.1 批量消费
```yaml
spring:
  kafka:
    listener:
      type: batch
    consumer:
      max-poll-records: 100
```

```java
@KafkaListener(topics = "mongo.changes")
public void consume(List<String> messages) {
    messages.forEach(message -> {
        // 处理每条消息
    });
}
```

### 6.2 错误处理与重试

```java
@Bean
public RetryTopicConfiguration retryTopicConfig(KafkaTemplate<String, String> template) {
    return RetryTopicConfigurationBuilder
        .newInstance()
        .fixedBackOff(3000)
        .maxAttempts(3)
        .create(template);
}
```

### 6.3 监控与指标

```java
@Bean
public KafkaListenerEndpointRegistry kafkaListenerEndpointRegistry() {
    return new KafkaListenerEndpointRegistry();
}

// 获取消费者指标
MeterRegistry meterRegistry = new SimpleMeterRegistry();
ConsumerFactory<String, String> consumerFactory = new DefaultKafkaConsumerFactory<>(consumerProps());
MicrometerConsumerListener<String, String> metricsListener = 
    new MicrometerConsumerListener<>(meterRegistry);
consumerFactory.addListener(metricsListener);
```

## 7. 生产环境注意事项

1. **安全性**：
   - 使用 SSL 加密 Kafka 和 MongoDB 连接
   - 配置适当的 ACL 权限

2. **性能**：
   - 调整 Kafka 分区数量以并行处理
   - 监控连接器延迟

3. **容错**：
   - 实现死信队列处理失败消息
   - 定期检查消费者偏移量

4. **数据一致性**：
   - 考虑使用事务处理关键业务逻辑
   - 实现幂等性消费者

这个方案提供了从 MongoDB 变更捕获到 Spring Boot 应用处理的完整流程，您可以根据具体业务需求进行调整和扩展。