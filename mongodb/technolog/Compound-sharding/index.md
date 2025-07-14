[返回](/mongodb/technolog/index)

# MongoDB 复合分片详解

复合分片(Compound Sharding)是MongoDB中一种高级的分片策略，它允许您使用多个字段的组合作为分片键，从而提供更灵活的数据分布方式。

## 什么是复合分片

复合分片是指使用多个字段的组合作为分片键。与单一字段分片键相比，复合分片键可以提供：
- 更均衡的数据分布
- 更好的查询性能
- 更灵活的数据局部性控制

## 创建复合分片键

### 基本语法

```javascript
sh.shardCollection("<database>.<collection>", { <field1>: <direction>, <field2>: <direction>, ... })
```

### 示例

假设我们有一个`orders`集合，想要以`customerId`和`orderDate`作为复合分片键：

```javascript
sh.shardCollection("shop.orders", { "customerId": 1, "orderDate": 1 })
```

## 复合分片键的特性

1. **字段顺序很重要**：第一个字段对数据分布影响最大
2. **方向性**：每个字段可以指定升序(1)或降序(-1)
3. **不可变性**：分片键一旦设置就不能更改

## 查询性能影响

复合分片键可以优化特定查询模式：

```javascript
// 高效查询 - 使用分片键前缀
db.orders.find({ "customerId": "C12345" })

// 高效查询 - 使用完整分片键
db.orders.find({ 
  "customerId": "C12345",
  "orderDate": { $gte: ISODate("2023-01-01") }
})
```

## 分片状态查看

查看分片状态：

```javascript
sh.status()
```

示例输出：
```
```
--- Sharding Status --- 
  sharding version: {
    "_id" : 1,
    "minCompatibleVersion" : 5,
    "currentVersion" : 6,
    "clusterId" : ObjectId("63d5a9c8f1a9b3a7c0d3e5f2")
  }
  shards:
    {  "_id" : "shard0000",  "host" : "shard00/localhost:27018",  "state" : 1 }
    {  "_id" : "shard0001",  "host" : "shard01/localhost:27019",  "state" : 1 }
  databases:
    {  "_id" : "shop",  "primary" : "shard0000",  "partitioned" : true,  "version" : {  "uuid" : UUID("a3b1c8d0-e5f2-4a7c-8d3e-5f2a7c0d3e5f"),  "lastMod" : 1 } }
      shop.orders
        shard key: { "customerId" : 1, "orderDate" : 1 }
        unique: false
        balancing: true
        chunks:
          shard0000    3
          shard0001    2
```
```

## 数据分布示例

查看数据分布情况：

```javascript
db.orders.getShardDistribution()
```

示例输出：
```
```
Collection shop.orders is sharded with shard key { "customerId" : 1, "orderDate" : 1 }

Shard shard0000 at shard00/localhost:27018
 data : 1.92GB docs : 1250000 chunks : 3
 estimated data per chunk : 640MB
 estimated docs per chunk : 416666

Shard shard0001 at shard01/localhost:27019
 data : 1.25GB docs : 850000 chunks : 2
 estimated data per chunk : 625MB
 estimated docs per chunk : 425000

Totals
 data : 3.17GB docs : 2100000 chunks : 5
 Shard shard0000 contains 59.52% data, 59.52% docs in cluster
 Shard shard0001 contains 40.48% data, 40.48% docs in cluster
```

## 复合分片键选择策略
```
| 策略                 | 优点            | 缺点            | 适用场景 |
|-----                -|-----           -|-----         -|----------|
| 高基数字段+低基数字段   | 分布均匀，查询灵活 | 可能产生较多块   | 大多数通用场景 |
| 单调递增字段+高基数字段 | 避免热点，分布均匀 | 范围查询可能跨分片 | 时间序列数据 |
| 两个高基数字段        | 最佳分布          | 可能增加查询复杂度 | 需要极致扩展性的场景 |
```
## 最佳实践

1. 选择查询最常用的字段组合
2. 确保前缀字段具有足够高的基数
3. 避免完全单调递增的复合键
4. 考虑数据局部性需求

## 限制

1. 分片键最大512字节
2. 分片键一旦设置不能更改
3. 空值的字段不能用作分片键的一部分