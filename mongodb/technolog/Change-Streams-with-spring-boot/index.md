[返回](/mongodb/technolog/index)

# 在Spring Boot中实现MongoDB变更流实时通知和提醒

在Spring Boot中集成MongoDB变更流(Change Streams)来实现实时通知和提醒功能，可以通过以下方式实现：

## 1. 基本配置

首先确保你的Spring Boot项目已经配置了MongoDB依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

## 2. 使用MongoTemplate实现变更流监听

```java
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoCursor;
import com.mongodb.client.model.changestream.ChangeStreamDocument;
import org.bson.Document;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.stereotype.Component;

@Component
public class OrderChangeListener {

    private final MongoTemplate mongoTemplate;
    
    public OrderChangeListener(MongoTemplate mongoTemplate) {
        this.mongoTemplate = mongoTemplate;
        startWatchingOrderChanges();
    }
    
    private void startWatchingOrderChanges() {
        new Thread(() -> {
            MongoCollection<Document> collection = mongoTemplate.getCollection("orders");
            
            // 监听orders集合的变更，特别是status字段的变化
            MongoCursor<ChangeStreamDocument<Document>> cursor = collection.watch()
                .fullDocument(FullDocument.UPDATE_LOOKUP)
                .filter(Filters.in("operationType", 
                    Arrays.asList("insert", "update", "replace")))
                .iterator();
            
            while (cursor.hasNext()) {
                ChangeStreamDocument<Document> change = cursor.next();
                handleChangeEvent(change);
            }
        }).start();
    }
    
    private void handleChangeEvent(ChangeStreamDocument<Document> change) {
        // 处理插入操作
        if ("insert".equals(change.getOperationTypeString())) {
            Document fullDocument = change.getFullDocument();
            System.out.println("新订单创建: " + fullDocument.toJson());
            sendNotification("新订单", fullDocument.getString("orderId"));
        }
        
        // 处理更新操作
        if ("update".equals(change.getOperationTypeString())) {
            Document updateDescription = change.getUpdateDescription();
            if (updateDescription != null && 
                updateDescription.containsKey("updatedFields") && 
                updateDescription.get("updatedFields", Document.class).containsKey("status")) {
                
                String newStatus = updateDescription.get("updatedFields", Document.class)
                    .getString("status");
                String orderId = change.getDocumentKey().getObjectId("_id").toString();
                
                System.out.println("订单状态更新: " + orderId + " -> " + newStatus);
                sendStatusChangeNotification(orderId, newStatus);
            }
        }
    }
    
    private void sendNotification(String title, String orderId) {
        // 实现通知逻辑，可以是邮件、短信、WebSocket等
        System.out.println("发送通知: " + title + " - 订单ID: " + orderId);
    }
    
    private void sendStatusChangeNotification(String orderId, String newStatus) {
        // 根据状态发送不同通知
        switch (newStatus) {
            case "paid":
                System.out.println("订单已支付通知: " + orderId);
                break;
            case "shipped":
                System.out.println("订单已发货通知: " + orderId);
                break;
            case "completed":
                System.out.println("订单已完成通知: " + orderId);
                break;
        }
    }
}
```

## 3. 使用Spring Data MongoDB的@ChangeStream注解（更简洁的方式）

Spring Data MongoDB提供了更简洁的注解方式来监听变更：

```java
import org.springframework.data.mongodb.core.messaging.*;
import org.springframework.stereotype.Service;

@Service
public class OrderNotificationService {

    @ChangeStream(
        target = Order.class,
        filter = "{'$match': {'operationType': {'$in': ['insert', 'update']}}}"
    )
    public void onOrderChange(ChangeStreamEvent<Order> event) {
        Order order = event.getBody();
        String operationType = event.getOperationType();
        
        if ("insert".equals(operationType)) {
            System.out.println("新订单: " + order.getId());
            sendNewOrderNotification(order);
        } else if ("update".equals(operationType)) {
            System.out.println("订单更新: " + order.getId());
            if (order.getStatus() != null) {
                sendStatusNotification(order.getId(), order.getStatus());
            }
        }
    }
    
    // 通知方法实现...
}
```

## 4. 整合WebSocket实现实时前端通知

如果需要将变更实时推送到前端，可以结合WebSocket：

```java
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Component;

@Component
public class RealTimeNotificationService {

    private final SimpMessagingTemplate messagingTemplate;
    
    public RealTimeNotificationService(SimpMessagingTemplate messagingTemplate) {
        this.messagingTemplate = messagingTemplate;
    }
    
    public void sendRealTimeNotification(String userId, String message) {
        messagingTemplate.convertAndSendToUser(
            userId, 
            "/queue/notifications", 
            new NotificationMessage(message)
        );
    }
    
    // 在变更监听器中调用此方法
}
```

## 5. 完整的订单状态变更通知示例

```java
import org.springframework.data.mongodb.core.ChangeStreamEvent;
import org.springframework.data.mongodb.core.ChangeStreamOptions;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.messaging.ChangeStreamRequest;
import org.springframework.data.mongodb.core.messaging.MessageListenerContainer;
import org.springframework.stereotype.Service;

@Service
public class OrderStatusChangeService {

    private final MongoTemplate mongoTemplate;
    private final MessageListenerContainer container;
    
    public OrderStatusChangeService(MongoTemplate mongoTemplate, 
                                  MessageListenerContainer container) {
        this.mongoTemplate = mongoTemplate;
        this.container = container;
        startListeningToStatusChanges();
    }
    
    private void startListeningToStatusChanges() {
        ChangeStreamOptions options = ChangeStreamOptions.builder()
            .filter("{'$match': {'operationType': 'update', "
                + "'updateDescription.updatedFields.status': {'$exists': true}}}")
            .build();
        
        ChangeStreamRequest<Order> request = ChangeStreamRequest.builder()
            .collection("orders")
            .publishTo(this::handleStatusChange)
            .options(options)
            .build();
        
        container.register(request, Order.class);
    }
    
    private void handleStatusChange(ChangeStreamEvent<Order> event) {
        Order order = event.getBody();
        String previousStatus = getPreviousStatus(event);
        String newStatus = order.getStatus();
        
        System.out.printf("订单状态变更: %s -> %s%n", previousStatus, newStatus);
        
        // 根据状态变更发送不同通知
        if ("paid".equals(newStatus)) {
            sendPaymentNotification(order);
        } else if ("shipped".equals(newStatus)) {
            sendShippingNotification(order);
        }
    }
    
    private String getPreviousStatus(ChangeStreamEvent<Order> event) {
        // 从变更事件中获取之前的状态
        // 实际实现可能需要从updateDescription中解析
        return "unknown";
    }
    
    // 具体的通知方法实现...
}
```

## 6. 配置变更流监听器自动启动

确保在应用启动时自动开始监听变更：

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class ChangeStreamInitializer implements CommandLineRunner {

    private final OrderChangeListener orderChangeListener;
    
    public ChangeStreamInitializer(OrderChangeListener orderChangeListener) {
        this.orderChangeListener = orderChangeListener;
    }
    
    @Override
    public void run(String... args) throws Exception {
        // 依赖注入时已经启动，这里不需要额外操作
    }
}
```

## 注意事项

1. **性能考虑**：变更流会占用数据库资源，高并发场景需要谨慎使用
2. **错误处理**：需要实现重试机制处理网络中断等情况
3. **集群环境**：确保你的MongoDB是副本集或分片集群配置
4. **资源释放**：应用关闭时需要正确关闭变更流监听
5. **事务支持**：MongoDB 4.0+支持事务，变更流可以监听事务中的操作

通过以上方式，你可以在Spring Boot应用中实现对MongoDB数据变更的实时监听，并根据业务需求发送各种通知和提醒。