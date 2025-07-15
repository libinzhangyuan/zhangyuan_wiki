[返回](/mongodb/technolog/change-stream-disconnect-ctl-restore/index)


# Spring Boot中使用Spring Data MongoDB处理变更流网络中断恢复

是的，在使用Spring Boot的`spring-boot-starter-data-mongodb`处理变更流(Change Stream)时，**必须考虑网络中断后的恢复机制**，否则可能会导致数据变更事件丢失或重复处理。

## 为什么需要处理网络中断恢复？

1. **生产环境网络不稳定**：云环境、容器编排等都可能导致短暂网络中断
2. **服务重启**：应用升级或故障重启后需要继续处理变更
3. **MongoDB维护**：数据库维护期间连接会中断

## Spring Data MongoDB中的变更流恢复机制

Spring Data MongoDB提供了几种处理中断恢复的方式：

### 1. 使用Resume Token恢复

这是最精确的恢复方式，通过存储和恢复resume token来实现：

```java
@Configuration
public class MongoChangeStreamConfig {

    @Autowired
    private MongoTemplate mongoTemplate;
    
    private BsonValue lastResumeToken;
    
    @PostConstruct
    public void startChangeStream() {
        MatchOperation matchStage = Aggregation.match(
            Criteria.where("operationType").in("insert", "update", "replace")
        );
        
        ChangeStreamOptions options = ChangeStreamOptions.builder()
            .resumeToken(lastResumeToken)
            .returnFullDocumentOnUpdate()
            .build();
        
        mongoTemplate.changeStream("orders", 
            Aggregation.newAggregation(matchStage), 
            options, 
            Document.class)
            .forEach(event -> {
                System.out.println("收到变更事件: " + event);
                // 存储resume token
                this.lastResumeToken = event.getResumeToken();
                // 处理业务逻辑...
            });
    }
}
```

### 2. 使用时间戳恢复

当没有保存resume token时，可以从特定时间点恢复：

```java
ChangeStreamOptions options = ChangeStreamOptions.builder()
    .startAtOperationTime(new BsonTimestamp(System.currentTimeMillis() / 1000, 1))
    .build();
```

## 完整的网络中断恢复实现示例

```java
@Service
public class OrderChangeStreamService {

    @Autowired
    private MongoTemplate mongoTemplate;
    
    @Value("${spring.data.mongodb.database}")
    private String dbName;
    
    private static final String COLLECTION = "orders";
    private static final String RESUME_TOKEN_KEY = "changeStreamResumeToken";
    
    // 存储resume token到MongoDB
    private void saveResumeToken(BsonValue token) {
        Document filter = new Document("_id", RESUME_TOKEN_KEY);
        Document update = new Document("$set", 
            new Document("token", token).append("collection", COLLECTION));
        mongoTemplate.getCollection("changeStreamState")
            .updateOne(filter, update, new UpdateOptions().upsert(true));
    }
    
    // 从MongoDB获取resume token
    private BsonValue getResumeToken() {
        Document doc = mongoTemplate.getCollection("changeStreamState")
            .find(new Document("_id", RESUME_TOKEN_KEY))
            .first();
        return doc != null ? doc.get("token", BsonValue.class) : null;
    }
    
    @PostConstruct
    public void startChangeStreamWithRetry() {
        Executors.newSingleThreadExecutor().execute(() -> {
            while (true) {
                try {
                    startChangeStream();
                } catch (Exception e) {
                    System.err.println("变更流异常，5秒后重试: " + e.getMessage());
                    try {
                        Thread.sleep(5000);
                    } catch (InterruptedException ie) {
                        Thread.currentThread().interrupt();
                        return;
                    }
                }
            }
        });
    }
    
    private void startChangeStream() {
        BsonValue resumeToken = getResumeToken();
        
        ChangeStreamOptions options = ChangeStreamOptions.builder()
            .resumeToken(resumeToken)
            .fullDocumentLookup(FullDocument.UPDATE_LOOKUP)
            .build();
        
        List<Bson> pipeline = singletonList(
            Aggregates.match(Filters.in("operationType", 
                "insert", "update", "replace", "delete")));
        
        try {
            mongoTemplate.getCollection(COLLECTION)
                .watch(pipeline)
                .withOptions(options)
                .forEach(event -> {
                    System.out.println("处理变更事件: " + event);
                    saveResumeToken(event.getResumeToken());
                    processChangeEvent(event);
                });
        } catch (MongoSocketException e) {
            throw new RuntimeException("网络中断", e);
        }
    }
    
    private void processChangeEvent(ChangeStreamDocument<Document> event) {
        // 实际的业务处理逻辑
    }
}
```

## 不同恢复策略对比

```
| 恢复方式               | 实现复杂度 | 数据精确度 | 资源消耗 | 适用场景                     |
|------------------------|------------|------------|----------|----------------------------|
| resumeToken           | 中         | 高(精确)    | 低       | 常规生产环境(推荐)          |
| operationTime         | 低         | 中(可能重复)| 低       | 简单场景，允许少量重复      |
| 全量同步+增量         | 高         | 高         | 高       | 数据一致性要求极高的场景    |
| 不处理恢复            | 低         | 低(会丢失)  | 低       | 仅用于开发测试环境          |
```

## 最佳实践建议

1. **持久化存储resumeToken**：
   - 使用MongoDB本身存储token
   - 或使用Redis等快速存储
   - 确保存储是原子操作

2. **错误处理和重试机制**：
   ```java
   @Retryable(value = {MongoSocketException.class}, 
              maxAttempts = 5,
              backoff = @Backoff(delay = 1000, multiplier = 2))
   public void startChangeStream() {
       // 变更流启动逻辑
   }
   ```

3. **监控和告警**：
   - 监控变更流延迟
   - 设置长时间未收到事件的告警

4. **考虑使用Spring Cloud Stream**：
   对于复杂场景，可以结合Spring Cloud Stream实现更健壮的管道：
   ```java
   @Bean
   public Supplier<Flux<ChangeStreamDocument<Document>>> orderChangeStream() {
       return () -> Flux.create(sink -> {
           ChangeStreamOptions options = // 配置选项;
           mongoTemplate.getCollection("orders")
               .watch().withOptions(options)
               .forEach(sink::next);
       });
   }
   ```

5. **集群环境处理**：
   - 在多个实例运行时，确保只有一个实例处理变更流
   - 可以使用Spring Integration的Leader选举功能

通过以上方式，在Spring Boot应用中可以实现健壮的MongoDB变更流处理，能够优雅地处理网络中断和恢复。