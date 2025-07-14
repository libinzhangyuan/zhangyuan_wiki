[返回](/mongodb/technolog/index)

# MongoDB 变更流(Change Streams)介绍

变更流是MongoDB提供的一个功能，允许应用程序实时获取数据库中的变更事件。它类似于关系型数据库中的触发器或消息队列，但更加灵活和强大。

## 变更流的基本概念

变更流可以监听以下类型的变更：
- 插入(insert)操作
- 更新(update)操作
- 替换(replace)操作
- 删除(delete)操作
- 集合的删除(drop)操作
- 数据库的删除(dropDatabase)操作
- 重命名(rename)操作
- 修改集合结构(invalidate)操作

## 如何使用变更流

### 基本语法

```javascript
const changeStream = db.collection.watch([pipeline], [options]);
```

### 参数说明

| 参数 | 类型 | 说明 |
|------|------|------|
| pipeline | array | 可选，用于过滤或修改变更事件的聚合管道 |
| options | document | 可选，配置变更流行为的选项 |

## 示例

### 示例1：监听集合的所有变更

```javascript
// 连接到MongoDB
const MongoClient = require('mongodb').MongoClient;
const client = new MongoClient('mongodb://localhost:27017');

async function watchChanges() {
    await client.connect();
    const collection = client.db('test').collection('users');
    
    const changeStream = collection.watch();
    
    changeStream.on('change', (change) => {
        console.log('检测到变更:', change);
    });
}

watchChanges();
```

可能的返回结果：

```
检测到变更: {
  _id: { _data: '8260B6B9C5000000012B022C0100296E5A1004...' },
  operationType: 'insert',
  clusterTime: Timestamp({ t: 1625067205, i: 1 }),
  fullDocument: {
    _id: ObjectId("60b6b9c5e6d8a1e5d8f7d8a1"),
    name: '张三',
    age: 25,
    email: 'zhangsan@example.com'
  },
  ns: { db: 'test', coll: 'users' },
  documentKey: { _id: ObjectId("60b6b9c5e6d8a1e5d8f7d8a1") }
}
```

### 示例2：只监听特定类型的变更

```javascript
const pipeline = [
    { $match: { operationType: { $in: ['insert', 'delete'] } } }
];

const changeStream = collection.watch(pipeline);
```

### 示例3：监听特定字段的变更

```javascript
const pipeline = [
    { $match: { 'updateDescription.updatedFields.age': { $exists: true } } }
];

const changeStream = collection.watch(pipeline);
```

可能的返回结果：

```
{
  _id: { _data: '8260B6B9C6000000012B022C0100296E5A1004...' },
  operationType: 'update',
  clusterTime: Timestamp({ t: 1625067206, i: 1 }),
  ns: { db: 'test', coll: 'users' },
  documentKey: { _id: ObjectId("60b6b9c5e6d8a1e5d8f7d8a1") },
  updateDescription: {
    updatedFields: { age: 26 },
    removedFields: []
  }
}
```

## 变更流选项

常用选项：

| 选项 | 类型 | 说明 |
|------|------|------|
| fullDocument | string | 设置为'updateLookup'会在更新操作时返回完整的文档 |
| resumeAfter | document | 从特定变更事件后恢复监听 |
| startAfter | document | 从特定变更事件后开始监听 |
| batchSize | integer | 每个批次返回的变更事件数量 |

### 示例：使用fullDocument选项

```javascript
const options = { fullDocument: 'updateLookup' };
const changeStream = collection.watch([], options);
```

可能的返回结果：

```
{
  _id: { _data: '8260B6B9C7000000012B022C0100296E5A1004...' },
  operationType: 'update',
  clusterTime: Timestamp({ t: 1625067207, i: 1 }),
  fullDocument: {
    _id: ObjectId("60b6b9c5e6d8a1e5d8f7d8a1"),
    name: '张三',
    age: 26,
    email: 'zhangsan@example.com'
  },
  ns: { db: 'test', coll: 'users' },
  documentKey: { _id: ObjectId("60b6b9c5e6d8a1e5d8f7d8a1") },
  updateDescription: {
    updatedFields: { age: 26 },
    removedFields: []
  }
}
```

## 变更流的恢复

变更流支持断点续传，可以使用resumeToken来恢复监听：

```javascript
// 获取最后一个变更事件的resumeToken
const resumeToken = changeStream.resumeToken;

// 使用resumeToken创建新的变更流
const newChangeStream = collection.watch([], { resumeAfter: resumeToken });
```

## 变更流的关闭

```javascript
// 关闭变更流
changeStream.close();
```

## 使用场景

1. **实时数据分析**：当数据变更时立即进行分析
2. **缓存同步**：保持缓存与数据库同步
3. **事件驱动架构**：触发下游系统处理
4. **审计日志**：记录所有数据变更

## 注意事项

1. 变更流需要MongoDB副本集或分片集群
2. 变更流使用oplog，所以会受到oplog大小的限制
3. 长时间不处理的变更流可能会导致oplog被覆盖
4. 变更流不保证顺序，特别是在分片环境中

希望这个介绍对您有所帮助！如果需要更具体的示例或说明，请随时提问。