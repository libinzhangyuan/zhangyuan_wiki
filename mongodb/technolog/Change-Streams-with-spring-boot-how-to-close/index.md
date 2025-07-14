[返回](/mongodb/technolog/index)

# 在Spring Boot中正确关闭MongoDB变更流监听

在Spring Boot应用中，为了确保应用关闭时能正确关闭MongoDB变更流监听，避免资源泄漏，可以通过以下几种方式实现：

## 1. 使用DisposableBean接口

```java
import org.springframework.beans.factory.DisposableBean;
import org.springframework.stereotype.Component;
import com.mongodb.client.MongoCursor;

@Component
public class OrderChangeListener implements DisposableBean {
    
    private volatile boolean running = true;
    private MongoCursor<ChangeStreamDocument<Document>> cursor;
    
    public void startListening() {
        new Thread(() -> {
            MongoCollection<Document> collection = mongoTemplate.getCollection("orders");
            cursor = collection.watch().iterator();
            
            while (running && cursor.hasNext()) {
                ChangeStreamDocument<Document> change = cursor.next();
                handleChange(change);
            }
        }).start();
    }
    
    @Override
    public void destroy() throws Exception {
        running = false;
        if (cursor != null) {
            cursor.close(); // 关闭变更流游标
            System.out.println("MongoDB变更流监听已关闭");
        }
    }
    
    // 其他方法...
}
```

## 2. 使用@PreDestroy注解

```java
import javax.annotation.PreDestroy;

@Component
public class OrderChangeListener {
    
    private MongoCursor<ChangeStreamDocument<Document>> cursor;
    
    @PreDestroy
    public void cleanUp() {
        if (cursor != null) {
            cursor.close();
            System.out.println("变更流监听器已清理");
        }
    }
    
    // 其他方法...
}
```

## 3. 使用Spring Data MongoDB的MessageListenerContainer

如果你使用Spring Data的高级API，可以这样管理：

```java
import org.springframework.data.mongodb.core.messaging.MessageListenerContainer;
import org.springframework.context.SmartLifecycle;

@Service
public class OrderChangeService implements SmartLifecycle {
    
    private final MessageListenerContainer container;
    private volatile boolean running = false;
    
    public OrderChangeService(MessageListenerContainer container) {
        this.container = container;
    }
    
    @Override
    public void start() {
        if (!running) {
            container.start();
            running = true;
        }
    }
    
    @Override
    public void stop() {
        if (running) {
            container.stop();
            running = false;
        }
    }
    
    @Override
    public boolean isRunning() {
        return running;
    }
    
    // 其他必要的方法...
}
```

## 4. 完整示例：结合Spring管理生命周期

```java
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextClosedEvent;
import org.springframework.stereotype.Component;

@Component
public class OrderChangeListener implements ApplicationListener<ContextClosedEvent> {
    
    private final MongoTemplate mongoTemplate;
    private MongoCursor<ChangeStreamDocument<Document>> cursor;
    private Thread listenerThread;
    private volatile boolean running = false;
    
    public OrderChangeListener(MongoTemplate mongoTemplate) {
        this.mongoTemplate = mongoTemplate;
        startListening();
    }
    
    public void startListening() {
        running = true;
        listenerThread = new Thread(() -> {
            MongoCollection<Document> collection = mongoTemplate.getCollection("orders");
            cursor = collection.watch().iterator();
            
            while (running && cursor.hasNext()) {
                try {
                    ChangeStreamDocument<Document> change = cursor.next();
                    handleChange(change);
                } catch (Exception e) {
                    if (running) {
                        // 处理异常并可能重试
                        System.err.println("变更流监听异常: " + e.getMessage());
                    }
                }
            }
        });
        
        listenerThread.setName("MongoDB-ChangeStream-Listener");
        listenerThread.start();
    }
    
    @Override
    public void onApplicationEvent(ContextClosedEvent event) {
        shutdown();
    }
    
    public void shutdown() {
        running = false;
        
        // 关闭游标
        if (cursor != null) {
            try {
                cursor.close();
            } catch (Exception e) {
                System.err.println("关闭变更流游标时出错: " + e.getMessage());
            }
        }
        
        // 中断监听线程
        if (listenerThread != null && listenerThread.isAlive()) {
            listenerThread.interrupt();
            try {
                listenerThread.join(1000); // 等待线程终止
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
        
        System.out.println("MongoDB变更流监听器已安全关闭");
    }
    
    // 其他业务方法...
}
```

## 5. 使用Spring Boot的Graceful Shutdown

在application.properties中配置优雅关闭：

```properties
# 启用优雅关闭
server.shutdown=graceful
# 设置等待时间(秒)
spring.lifecycle.timeout-per-shutdown-phase=30s
```

然后结合@PreDestroy使用：

```java
import org.springframework.beans.factory.annotation.Autowired;
import javax.annotation.PreDestroy;
import java.util.concurrent.ExecutorService;

@Component
public class OrderChangeListener {
    
    @Autowired
    private ExecutorService taskExecutor;
    
    private volatile boolean running = true;
    
    @PreDestroy
    public void onDestroy() {
        running = false;
        taskExecutor.shutdownNow(); // 如果使用线程池
        System.out.println("变更流监听器正在关闭...");
    }
    
    // 其他方法...
}
```

## 6. 最佳实践建议

1. **资源清理顺序**：
   - 先设置运行标志为false
   - 然后关闭MongoDB游标
   - 最后中断/停止监听线程

2. **异常处理**：
   ```java
   @PreDestroy
   public void cleanUp() {
       try {
           if (cursor != null) {
               cursor.close();
           }
       } catch (Exception e) {
           System.err.println("关闭变更流时出错: " + e.getMessage());
       } finally {
           running = false;
       }
   }
   ```

3. **监听线程管理**：
   ```java
   private Thread listenerThread;
   
   public void start() {
       listenerThread = new Thread(this::listen);
       listenerThread.setDaemon(false); // 设为非守护线程
       listenerThread.start();
   }
   
   public void stop() {
       if (listenerThread != null) {
           listenerThread.interrupt();
       }
   }
   ```

4. **使用Spring的SmartLifecycle**（更精细控制）：
   ```java
   @Component
   public class ChangeStreamLifecycle implements SmartLifecycle {
       
       private volatile boolean running = false;
       
       @Override
       public void start() {
           running = true;
           // 启动监听逻辑
       }
       
       @Override
       public void stop() {
           running = false;
           // 关闭资源
       }
       
       @Override
       public boolean isRunning() {
           return running;
       }
       
       @Override
       public int getPhase() {
           return Integer.MAX_VALUE; // 最后关闭
       }
   }
   ```

通过以上方法，你可以确保Spring Boot应用在关闭时能够正确、优雅地关闭MongoDB变更流监听，避免资源泄漏和数据不一致的问题。