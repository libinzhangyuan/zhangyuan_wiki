[返回](/mongodb/technolog/index)

# MongoDB 分片与分桶详解

## 分片 (Sharding)

分片是 MongoDB 中的水平数据分区方法，它将大型数据集分布在多个服务器或分片上，每个分片只保存数据集的一部分。

### 分片核心概念

1. **分片键 (Shard Key)**：决定数据如何在分片间分布的字段
2. **分片 (Shard)**：存储部分数据的 MongoDB 实例
3. **配置服务器 (Config Server)**：存储集群元数据
4. **查询路由器 (Mongos)**：路由客户端请求到适当分片

### 分片示例

假设我们有一个 `orders` 集合，按 `customer_id` 分片：

```javascript
// 启用分片
sh.enableSharding("mydb")

// 选择分片键
sh.shardCollection("mydb.orders", { "customer_id": 1 })

// 插入一些数据
db.orders.insertMany([
  { "customer_id": "C1001", "amount": 150, "date": ISODate("2023-01-01") },
  { "customer_id": "C1002", "amount": 200, "date": ISODate("2023-01-02") },
  { "customer_id": "C1003", "amount": 75, "date": ISODate("2023-01-03") }
])

// 查看分片状态
sh.status()
```

返回结果示例：
```
```json
{
  "shardedCollections": {
    "mydb.orders": {
      "shardKey": { "customer_id": 1 },
      "chunks": [
        { "shard": "shard1", "min": { "customer_id": MinKey() }, "max": { "customer_id": "C1002" } },
        { "shard": "shard2", "min": { "customer_id": "C1002" }, "max": { "customer_id": MaxKey() } }
      ]
    }
  }
}
```
```

## 分桶 (Bucketing)

分桶是一种数据组织模式，将具有相似特征的文档分组到同一文档或"桶"中，通常与时间序列数据一起使用。

### 分桶示例

假设我们收集传感器数据，每分钟创建一个桶：

```javascript
// 创建分桶集合
db.createCollection("sensor_data")

// 插入分桶数据
db.sensor_data.insertOne({
  "sensor_id": "S001",
  "start_time": ISODate("2023-01-01T00:00:00Z"),
  "end_time": ISODate("2023-01-01T00:01:00Z"),
  "measurements": [
    { "timestamp": ISODate("2023-01-01T00:00:10Z"), "value": 23.5 },
    { "timestamp": ISODate("2023-01-01T00:00:20Z"), "value": 23.7 },
    { "timestamp": ISODate("2023-01-01T00:00:30Z"), "value": 23.6 }
  ],
  "stats": {
    "avg_value": 23.6,
    "max_value": 23.7,
    "min_value": 23.5
  }
})
```

查询分桶数据：
```javascript
db.sensor_data.find({
  "sensor_id": "S001",
  "start_time": { "$gte": ISODate("2023-01-01T00:00:00Z") },
  "end_time": { "$lt": ISODate("2023-01-01T00:02:00Z") }
})
```

返回结果示例：
```
```json
{
  "_id": ObjectId("5f8d8a7b8c7d6b5a4c3b2a1f"),
  "sensor_id": "S001",
  "start_time": ISODate("2023-01-01T00:00:00Z"),
  "end_time": ISODate("2023-01-01T00:01:00Z"),
  "measurements": [
    { "timestamp": ISODate("2023-01-01T00:00:10Z"), "value": 23.5 },
    { "timestamp": ISODate("2023-01-01T00:00:20Z"), "value": 23.7 },
    { "timestamp": ISODate("2023-01-01T00:00:30Z"), "value": 23.6 }
  ],
  "stats": {
    "avg_value": 23.6,
    "max_value": 23.7,
    "min_value": 23.5
  }
}
```


## 分片与分桶对比

```
| 特性                | 分片 (Sharding)                          | 分桶 (Bucketing)                        |
|---------------------|-----------------------------------------|----------------------------------------|
| 目的                | 水平扩展数据库                          | 优化数据组织方式                       |
| 数据分布            | 跨多个服务器                            | 在单个文档中分组                       |
| 适用场景            | 大型数据集                              | 时间序列或高频率写入数据               |
| 性能影响            | 提高读写吞吐量                          | 减少文档数量，提高查询效率             |
| 实现方式            | 通过MongoDB集群配置                     | 通过应用程序逻辑实现                   |
| 查询复杂度          | 可能需要跨分片查询                      | 通常查询更简单                         |
| 数据一致性          | 需要处理分布式一致性                    | 单文档原子性                           |
| 存储效率            | 可能因分片键选择不当导致不均匀分布      | 减少索引大小和文档开销                 |
```

