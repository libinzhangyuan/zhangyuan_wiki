[返回](/mongodb/technolog/change-stream-disconnect-ctl-restore/index)

# MongoDB变更流(Change Stream)网络中断后的恢复机制

MongoDB的变更流(Change Stream)允许应用程序实时监听数据库中的变更事件。当网络中断时，恢复机制至关重要，以确保不会丢失任何变更事件。

## 恢复机制核心原理

MongoDB变更流使用resume token来跟踪已经处理的事件位置。在网络中断后，可以通过以下方式恢复：

1. **使用resume token恢复**：存储最后处理的resume token，中断后从中断点继续
2. **设置startAtOperationTime**：从特定时间点开始恢复
3. **全量同步+增量变更**：极端情况下可以先做全量同步再监听变更

## 恢复机制示例

### 1. 基本恢复示例

```javascript
// 存储最后的resumeToken
let lastResumeToken = null;

const pipeline = [{ $match: { operationType: 'insert' } }];
const options = { fullDocument: 'updateLookup' };

// 第一次监听
const changeStream = db.collection('orders').watch(pipeline, options);

changeStream.on('change', (change) => {
  console.log('收到变更:', change);
  lastResumeToken = change._id; // 存储resumeToken
});

// 网络中断后恢复
function resumeChangeStream() {
  const resumeOptions = { resumeAfter: lastResumeToken };
  const newChangeStream = db.collection('orders').watch(pipeline, resumeOptions);
  
  newChangeStream.on('change', (change) => {
    console.log('恢复后收到变更:', change);
    lastResumeToken = change._id;
  });
}
```

### 2. 使用startAtOperationTime恢复

```javascript
const changeStream = db.collection('orders').watch([], {
  startAtOperationTime: new Timestamp(1625097600, 1) // 从特定时间点开始
});
```

## 恢复策略对比

```
| 恢复策略               | 优点                          | 缺点                          | 适用场景                     |
|------------------------|-----------------------------|-----------------------------|----------------------------|
| resumeAfter            | 精确恢复，不丢事件             | 需要持久化存储resumeToken     | 常规中断恢复                |
| startAtOperationTime   | 不需要存储token               | 可能重复或丢失部分事件         | 无法获取resumeToken时       |
| startAfter            | 从指定事件后开始(不含该事件)    | 需要存储token                 | 需要跳过已处理事件的场景     |
| 全量同步+增量          | 最可靠                        | 资源消耗大                    | 数据一致性要求极高的场景     |
```

## 实际应用示例

### 示例1：存储和恢复resumeToken

```javascript
// 假设我们有一个存储token的函数
async function saveResumeToken(token) {
  await db.collection('changeStreamState').updateOne(
    { _id: 'orders' },
    { $set: { lastResumeToken: token } },
    { upsert: true }
  );
}

// 获取存储的token
async function getResumeToken() {
  const doc = await db.collection('changeStreamState').findOne({ _id: 'orders' });
  return doc ? doc.lastResumeToken : null;
}

// 启动变更流
async function startChangeStream() {
  const resumeToken = await getResumeToken();
  const options = resumeToken ? { resumeAfter: resumeToken } : {};
  
  const changeStream = db.collection('orders').watch([], options);
  
  changeStream.on('change', async (change) => {
    console.log('变更事件:', change);
    await saveResumeToken(change._id);
    // 处理业务逻辑...
  });
  
  changeStream.on('error', (err) => {
    console.error('变更流错误:', err);
    // 错误处理逻辑...
  });
}
```

### 示例2：处理网络中断

```javascript
const { MongoNetworkError } = require('mongodb');

async function startChangeStreamWithRetry() {
  let changeStream;
  let attempt = 0;
  const maxAttempts = 5;
  
  const startStream = async () => {
    const resumeToken = await getResumeToken();
    const options = resumeToken ? { resumeAfter: resumeToken } : {};
    
    changeStream = db.collection('orders').watch([], options);
    
    changeStream.on('change', async (change) => {
      console.log('变更事件:', change);
      await saveResumeToken(change._id);
    });
    
    changeStream.on('error', (err) => {
      if (err instanceof MongoNetworkError) {
        console.log('网络错误，准备重试...');
        attempt++;
        if (attempt <= maxAttempts) {
          setTimeout(() => {
            changeStream.close();
            startStream();
          }, 1000 * attempt); // 指数退避
        }
      }
    });
  };
  
  await startStream();
}
```

## 最佳实践

1. **持久化存储resumeToken**：定期将resumeToken保存到持久化存储中
2. **错误处理**：监听error事件并实现重试逻辑
3. **心跳检测**：对于长时间没有变更的情况，可以添加心跳文档定期更新
4. **考虑使用MongoDB Atlas**：Atlas提供了更稳定的变更流服务

通过合理使用这些恢复机制，可以确保即使在网络不稳定的环境下，基于MongoDB变更流的应用也能保持数据的完整性和一致性。