[return](/mongodb/technolog/table-index/index)

# MongoDB TTL索引 (TTL Index) 介绍

TTL索引是MongoDB中一种特殊的单字段索引，它允许自动删除集合中的过期文档。这种索引对于管理临时数据如会话信息、日志、事件数据等非常有用。

## TTL索引的特点

1. **自动过期删除**：MongoDB会自动删除超过指定时间的文档
2. **后台执行**：过期检查在后台每60秒运行一次
3. **基于日期类型字段**：只能应用于日期类型的字段
4. **单字段索引**：只能是单字段索引，不能是复合索引

## 创建TTL索引

基本语法：
```javascript
db.collection.createIndex({ field: 1 }, { expireAfterSeconds: 3600 })
```

### 示例1：创建基本的TTL索引

```javascript
// 创建一个60秒后过期的索引
db.logs.createIndex({ "createdAt": 1 }, { expireAfterSeconds: 60 })
```

### 示例2：使用现有日期字段

假设我们有一个`events`集合，其中包含`lastModified`字段：

```javascript
db.events.createIndex({ "lastModified": 1 }, { expireAfterSeconds: 86400 })
```

## TTL索引工作原理

1. MongoDB会检查索引字段的日期值
2. 如果日期值早于当前时间减去`expireAfterSeconds`值
3. 则该文档会被标记为过期并从集合中删除

## 示例数据及效果

### 插入测试数据

```javascript
db.logs.insertMany([
    { "message": "Test 1", "createdAt": new Date() },
    { "message": "Test 2", "createdAt": new Date(Date.now() - 120000) }, // 2分钟前
    { "message": "Test 3", "createdAt": new Date(Date.now() - 30000) }   // 30秒前
])
```

### 查询初始数据

```javascript
db.logs.find().sort({ createdAt: 1 })
```

返回结果：
```
[
  {
    "_id": ObjectId("5f3c7a8b8e4a2e1a2c8b4567"),
    "message": "Test 2",
    "createdAt": ISODate("2023-08-20T08:58:00Z") // 2分钟前
  },
  {
    "_id": ObjectId("5f3c7a8b8e4a2e1a2c8b4569"),
    "message": "Test 3",
    "createdAt": ISODate("2023-08-20T09:59:30Z") // 30秒前
  },
  {
    "_id": ObjectId("5f3c7a8b8e4a2e1a2c8b4568"),
    "message": "Test 1",
    "createdAt": ISODate("2023-08-20T10:00:00Z") // 现在
  }
]
```

60秒后再次查询：
```javascript
db.logs.find().sort({ createdAt: 1 })
```

返回结果：
```
[
  {
    "_id": ObjectId("5f3c7a8b8e4a2e1a2c8b4569"),
    "message": "Test 3",
    "createdAt": ISODate("2023-08-20T09:59:30Z")
  },
  {
    "_id": ObjectId("5f3c7a8b8e4a2e1a2c8b4568"),
    "message": "Test 1",
    "createdAt": ISODate("2023-08-20T10:00:00Z")
  }
]
```

可以看到最早的一条记录（2分钟前的）已被自动删除。

## TTL索引的特殊用法

### 动态过期时间

可以在文档中设置不同的过期时间：

```javascript
db.logs.createIndex({ "expireAt": 1 }, { expireAfterSeconds: 0 })

// 插入文档时指定具体的过期时间点
db.logs.insertOne({
    "message": "Dynamic expiration",
    "expireAt": new Date(Date.now() + 3600000) // 1小时后过期
})
```

## 注意事项

1. TTL索引不能是复合索引
2. 如果字段不是日期类型或者不包含日期，文档不会过期
3. 删除操作由后台线程执行，可能会有延迟
4. 副本集中，TTL删除操作只在主节点执行
5. 对于大集合，删除操作可能会影响性能

## TTL索引 vs 普通索引

```
| 特性              | TTL索引               | 普通索引               |
|-------------------|-----------------------|-----------------------|
| 用途              | 自动删除过期文档       | 加速查询               |
| 字段类型限制      | 必须是日期类型         | 任意类型               |
| 索引类型          | 只能是单字段索引       | 可以是单字段或复合索引 |
| 后台操作          | 有后台线程定期检查     | 无特殊后台操作         |
| 存储影响          | 会实际删除文档         | 只影响查询性能         |
```

## 管理TTL索引

### 查看已有TTL索引

```javascript
db.logs.getIndexes()
```

返回结果示例：
```
[
  {
    "v": 2,
    "key": { "_id": 1 },
    "name": "_id_"
  },
  {
    "v": 2,
    "key": { "createdAt": 1 },
    "name": "createdAt_1",
    "expireAfterSeconds": 60
  }
]
```

### 修改TTL过期时间

```javascript
db.runCommand({
    "collMod": "logs",
    "index": {
        "keyPattern": { "createdAt": 1 },
        "expireAfterSeconds": 3600 // 改为1小时
    }
})
```

### 删除TTL索引

```javascript
db.logs.dropIndex("createdAt_1")
```

TTL索引是MongoDB中管理数据生命周期的强大工具，合理使用可以简化很多数据清理工作。