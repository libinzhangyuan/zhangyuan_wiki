[返回](/mongodb/technolog/index)

# MongoDB 范围分片 vs 哈希分片深度对比

在 MongoDB 分片集群中，范围分片(Ranged Sharding)和哈希分片(Hashed Sharding)是两种主要的分片策略，它们各有特点和适用场景。

## 核心概念对比

```
| 对比维度          | 范围分片                                      | 哈希分片                                      |
|-------------------|---------------------------------------------|---------------------------------------------|
| 数据分布原理      | 基于分片键值的自然范围进行连续分布            | 使用哈希函数将分片键值转换为哈希值后随机分布  |
| 查询性能特点      | 范围查询高效，可以定向访问特定分片            | 等值查询高效，但范围查询需要访问所有分片      |
| 数据局部性        | 保持数据在物理存储上的连续性                  | 数据完全分散，无局部性                      |
| 写入热点风险      | 对单调递增/递减的分片键容易出现热点           | 有效避免热点问题                            |
| 分片键选择        | 需要精心选择以避免数据倾斜                    | 对分片键的选择相对宽松                      |
| 典型应用场景      | 时间序列、有序数据、范围查询多的场景          | 高吞吐写入、随机读取、无范围查询需求的场景    |
```

## 技术实现对比

### 范围分片实现示例

```javascript
// 启用范围分片
sh.enableSharding("orders")
sh.shardCollection("orders.transactions", { "orderDate": 1 })

// 插入数据示例
db.transactions.insertMany([
  { "orderId": "1001", "orderDate": new Date("2023-01-01"), "amount": 150 },
  { "orderId": "1002", "orderDate": new Date("2023-01-02"), "amount": 200 },
  { "orderId": "1003", "orderDate": new Date("2023-01-03"), "amount": 99 }
])
```

查看分片分布：

```javascript
sh.status()
```

可能的返回结果：

```
```
sharding version: {
  "_id" : 1,
  "minCompatibleVersion" : 5,
  "currentVersion" : 6,
  "clusterId" : ObjectId("5f1a5d9f9c9d663b703f7f7a")
}
shards:
  { "_id" : "shard0000", "host" : "shard0000/localhost:27018" }
  { "_id" : "shard0001", "host" : "shard0001/localhost:27019" }
databases:
  { "_id" : "orders", "primary" : "shard0000", "partitioned" : true }
    orders.transactions
      shard key: { "orderDate" : 1 }
      chunks:
        shard0000    2
        shard0001    1
      { "orderDate" : { "$minKey" : 1 } } -->> { "orderDate" : ISODate("2023-01-02T00:00:00Z") } on : shard0000 
      { "orderDate" : ISODate("2023-01-02T00:00:00Z") } -->> { "orderDate" : ISODate("2023-01-03T00:00:00Z") } on : shard0000 
      { "orderDate" : ISODate("2023-01-03T00:00:00Z") } -->> { "orderDate" : { "$maxKey" : 1 } } on : shard0001
```
```

### 哈希分片实现示例

```javascript
// 启用哈希分片
sh.enableSharding("users")
sh.shardCollection("users.profiles", { "username": "hashed" })

// 插入数据示例
db.profiles.insertMany([
  { "username": "alice123", "email": "alice@example.com", "age": 28 },
  { "username": "bob456", "email": "bob@example.com", "age": 32 },
  { "username": "charlie789", "email": "charlie@example.com", "age": 25 }
])
```

查看分片分布：

```javascript
db.profiles.getShardDistribution()
```

可能的返回结果：

```
Shard shard0000 at localhost:27018
 data : 768B docs : 2 chunks : 1
 estimated data per chunk : 768B
 estimated docs per chunk : 2

Shard shard0001 at localhost:27019
 data : 384B docs : 1 chunks : 1
 estimated data per chunk : 384B
 estimated docs per chunk : 1

Totals
 data : 1.1KB docs : 3 chunks : 2
 Shard shard0000 contains 66.67% data, 66.67% docs in cluster
 Shard shard0001 contains 33.33% data, 33.33% docs in cluster
```

## 性能特征对比

### 查询性能对比表

```
| 查询类型          | 范围分片表现                          | 哈希分片表现                          |
|-------------------|-------------------------------------|-------------------------------------|
| 等值查询          | 需要定位到特定分片                    | 直接路由到目标分片(最高效)            |
| 范围查询          | 只需访问相关分片(高效)                | 必须查询所有分片(性能差)              |
| 排序查询          | 天然有序，性能好                      | 需要在内存中合并排序(性能较差)        |
| 聚合操作          | 可以在单个分片完成部分聚合            | 通常需要合并所有分片数据              |
| 地理空间查询      | 支持本地化优化                        | 效率较低                            |
```

## 选择策略指南

### 何时选择范围分片？

1. **时间序列数据**：如日志、传感器数据、交易记录
   ```javascript
   // 时间序列数据理想的分片方式
   sh.shardCollection("logs.entries", { "timestamp": 1 })
   ```

2. **需要范围查询的场景**：如按日期范围查询订单
   ```javascript
   // 范围查询示例
   db.orders.find({ 
     orderDate: { 
       $gte: new Date("2023-01-01"), 
       $lte: new Date("2023-01-31") 
     }
   })
   ```

3. **需要保持数据局部性的场景**：如地理空间数据

### 何时选择哈希分片？

1. **高吞吐写入场景**：如用户行为日志
   ```javascript
   // 避免写入热点的分片方式
   sh.shardCollection("analytics.events", { "eventId": "hashed" })
   ```

2. **无明显查询模式的场景**：如随机访问的用户数据
   ```javascript
   // 随机访问示例
   db.users.find({ username: "alice123" })
   ```

3. **分片键单调递增的场景**：如自增ID、时间戳
   ```javascript
   // 对单调递增的_id使用哈希分片
   sh.shardCollection("products.items", { "_id": "hashed" })
   ```

## 高级技巧

### 复合分片键策略

结合两种策略的优点：

```javascript
// 使用复合分片键：首先按地区范围分片，然后对用户ID哈希分片
sh.shardCollection("global.users", { "region": 1, "userId": "hashed" })
```

### 分片策略转换注意事项

1. 从哈希分片改为范围分片（或反之）需要：
   - 创建新集合
   - 导出原数据
   - 导入到新集合
   - 更新应用配置

2. 监控分片平衡状态：
   ```javascript
   // 检查集群平衡状态
   db.adminCommand({ balancerStatus: 1 })
   
   // 手动触发平衡轮次
   sh.startBalancer()
   ```

## 实际案例分析

### 案例1：电商平台订单系统

**需求特点**：
- 频繁按日期范围查询订单
- 订单ID单调递增

**解决方案**：
```javascript
// 使用复合分片键
sh.shardCollection("ecommerce.orders", { 
  "orderDate": 1,    // 范围分片用于日期查询
  "orderId": "hashed" // 哈希分片避免热点
})
```

### 案例2：IoT传感器数据平台

**需求特点**：
- 海量时间序列数据写入
- 主要按设备ID和时间范围查询

**解决方案**：
```javascript
// 按设备ID哈希分片，时间戳作为次要字段
sh.shardCollection("iot.sensorData", {
  "deviceId": "hashed",  // 均匀分布写入负载
  "timestamp": 1         // 保持时间序列局部性
})
```

选择合适的分片策略需要综合考虑数据模型、查询模式和写入模式。在实际生产中，通常需要结合性能测试来确定最佳策略。