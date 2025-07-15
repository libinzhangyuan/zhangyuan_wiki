[返回](/mongodb/technolog/index)


# MongoDB 检测表(集合)更改或新增数据的手段

在 MongoDB 中，检测集合(表)的更改或新增数据有几种常用方法，下面我将详细介绍并提供示例。

## 1. Change Streams (变更流)

这是 MongoDB 3.6+ 提供的实时监控集合变更的功能，类似于关系型数据库的触发器。

```javascript
// 监控 orders 集合的所有变更
const changeStream = db.orders.watch();

changeStream.on('change', (change) => {
  console.log(change);
});
```

可能的返回结果示例：
```
{
  _id: { _data: '8260...' },
  operationType: 'insert',
  clusterTime: Timestamp({ t: 1620000000, i: 1 }),
  fullDocument: {
    _id: ObjectId("5f8d..."),
    customer: '张三',
    amount: 100,
    date: ISODate("2023-05-01T00:00:00Z")
  },
  ns: { db: 'test', coll: 'orders' },
  documentKey: { _id: ObjectId("5f8d...") }
}
```

## 2. 使用时间戳字段查询新增数据

为文档添加时间戳字段，然后定期查询大于上次检查时间的数据。

```javascript
// 添加/更新文档时包含时间戳
db.orders.insertOne({
  customer: "李四",
  amount: 200,
  createdAt: new Date()
});

// 查询新增数据
const lastChecked = new Date("2023-05-01T00:00:00Z");
db.orders.find({
  createdAt: { $gt: lastChecked }
});
```

返回结果示例：
```
[
  {
    "_id": ObjectId("5f8d..."),
    "customer": "李四",
    "amount": 200,
    "createdAt": ISODate("2023-05-02T10:00:00Z")
  }
]
```

## 3. 使用 oplog (操作日志)

适用于副本集环境，可以监控所有数据库操作。

```javascript
// 查询 oplog
db.getSiblingDB("local").oplog.rs.find({
  ns: "test.orders",
  op: { $in: ["i", "u", "d"] } // i-插入, u-更新, d-删除
}).sort({ $natural: -1 }).limit(10);
```

## 4. 对比前后查询结果

定期查询并保存结果，与之前的结果进行对比。

```javascript
// 第一次查询
const previousData = db.orders.find().toArray();

// 一段时间后...
const currentData = db.orders.find().toArray();

// 找出新增文档
const newDocuments = currentData.filter(doc => 
  !previousData.some(p => p._id.equals(doc._id))
);
```

## 方法对比表

```markdown
| 方法               | 实时性 | 复杂度 | 适用场景                     | 备注                     |
|--------------------|--------|--------|------------------------------|--------------------------|
| Change Streams     | 高     | 中     | 需要实时监控变更             | 需要MongoDB 3.6+         |
| 时间戳查询         | 低     | 低     | 可以接受延迟的变更检测       | 需要文档有时间戳字段     |
| Oplog              | 高     | 高     | 副本集环境下的全量操作监控   | 需要访问local数据库权限  |
| 对比前后查询结果   | 低     | 中     | 小型数据集，不频繁的变更检测 | 可能消耗较多内存         |
```

## 最佳实践建议

1. 对于需要实时监控的场景，优先使用 Change Streams
2. 对于简单的新增检测，使用时间戳字段是最简单的方法
3. 在生产环境中，考虑使用专门的变更数据捕获(CDC)工具
4. 对于大量数据变更，注意监控性能影响

希望这些方法能帮助你有效地监控 MongoDB 集合的变更！