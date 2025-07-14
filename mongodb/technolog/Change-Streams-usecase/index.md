[返回](/mongodb/technolog/index)

# MongoDB 变更流(Change Streams)应用场景介绍

变更流是MongoDB提供的一个强大功能，它允许应用程序实时监听数据库中的变更事件。以下是变更流的典型应用场景：

## 1. 实时数据同步

当数据库中的数据发生变化时，可以实时同步到其他系统或缓存中。

```javascript
// 监听products集合的变更
const changeStream = db.products.watch();

changeStream.on('change', (change) => {
  console.log('检测到变更:', change);
  // 将变更同步到Redis缓存或Elasticsearch
});
```

可能的返回结果：
```
{
  _id: { _data: '82632B3...' },
  operationType: 'insert',
  clusterTime: Timestamp({ t: 1625097600, i: 1 }),
  fullDocument: {
    _id: ObjectId("60f3b1a5e9d8a9a8f8f8f8f8"),
    name: '新商品',
    price: 99.99,
    stock: 100
  },
  ns: { db: 'shop', coll: 'products' },
  documentKey: { _id: ObjectId("60f3b1a5e9d8a9a8f8f8f8f8") }
}
```

## 2. 实时通知和提醒

当特定数据变更时，触发通知或提醒。

```javascript
// 监听订单状态变更
const changeStream = db.orders.watch([
  { $match: { 'updateDescription.updatedFields.status': { $exists: true } } }
]);

changeStream.on('change', (change) => {
  if (change.updateDescription.updatedFields.status === 'shipped') {
    // 发送发货通知
    sendShippingNotification(change.documentKey._id);
  }
});
```

## 3. 数据审计和日志记录

记录所有数据变更用于审计目的。

```javascript
// 监听所有集合的变更并记录到审计日志
const changeStream = db.watch();

changeStream.on('change', (change) => {
  db.auditLogs.insertOne({
    eventTime: new Date(),
    operation: change.operationType,
    collection: change.ns.coll,
    documentId: change.documentKey._id,
    changes: change.updateDescription,
    user: change.fullDocument.lastModifiedBy
  });
});
```

## 4. 微服务间数据同步

在微服务架构中保持服务间数据一致性。

```javascript
// 服务A监听服务B的数据变更
const changeStream = db.customers.watch();

changeStream.on('change', (change) => {
  if (change.operationType === 'update') {
    // 同步更新本地缓存
    updateLocalCustomerCache(
      change.documentKey._id, 
      change.updateDescription.updatedFields
    );
  }
});
```

## 5. 实时数据分析

实时处理数据变更进行即时分析。

```javascript
// 实时统计销售额
let totalSales = 0;

const changeStream = db.orders.watch([
  { $match: { operationType: 'insert' } }
]);

changeStream.on('change', (change) => {
  totalSales += change.fullDocument.amount;
  console.log(`实时销售额: ${totalSales}`);
});
```

## 变更流事件类型对比

```
| 操作类型       | 描述                     | 包含的字段示例                     |
|----------------|--------------------------|------------------------------------|
| insert         | 文档插入                 | fullDocument, documentKey         |
| update         | 文档更新                 | updateDescription, documentKey    |
| replace        | 文档替换                 | fullDocument, documentKey         |
| delete         | 文档删除                 | documentKey                        |
| invalidate     | 变更流无效(如集合删除)   | 无特定字段                         |
| 其他操作       | 如创建索引等DDL操作      | 通常不会触发变更流                 |
```

## 使用注意事项

1. 变更流需要MongoDB副本集或分片集群环境
2. 需要考虑网络中断后的恢复机制
3. 高频率变更场景下需要注意性能影响
4. 可以使用resumeToken实现断点续传

变更流为构建响应式、实时的应用程序提供了强大支持，是现代应用开发中非常有价值的工具。