# 分片(Sharding) 详解

**分片(Sharding)** 是 MongoDB 实现水平扩展的核心机制。下面我将详细介绍 MongoDB 的分区实现。

## 分片的基本原理

MongoDB 通过将数据分散存储在多个服务器(分片)上来实现分区，每个分片存储数据的一个子集。这种架构允许系统突破单机资源限制。

## 分区关键组件

```
| 组件             | 作用                                                                 |
|------------------|----------------------------------------------------------------------|
| mongos           | 查询路由器，负责将客户端请求路由到正确的分片                         |
| Config Server    | 存储集群元数据和配置信息(必须部署为副本集)                           |
| Shard            | 实际存储数据的节点(每个分片建议部署为副本集以提高可用性)             |
```

## 分区设置完整示例

### 1. 启动配置服务器副本集
```javascript
// 启动配置服务器(通常需要3个节点)
mongod --configsvr --replSet configReplSet --port 27019 --dbpath /data/configdb
```

### 2. 初始化配置服务器
```javascript
rs.initiate(
  {
    _id: "configReplSet",
    configsvr: true,
    members: [
      { _id: 0, host: "cfg1.example.net:27019" },
      { _id: 1, host: "cfg2.example.net:27019" },
      { _id: 2, host: "cfg3.example.net:27019" }
    ]
  }
)
```

### 3. 启动分片服务器
```javascript
// 启动分片1的副本集
mongod --shardsvr --replSet shardReplA --port 27018 --dbpath /data/shardA

// 初始化分片1副本集
rs.initiate(
  {
    _id: "shardReplA",
    members: [
      { _id: 0, host: "shardA1.example.net:27018" },
      { _id: 1, host: "shardA2.example.net:27018" },
      { _id: 2, host: "shardA3.example.net:27018", arbiterOnly: true }
    ]
  }
)
```

### 4. 启动 mongos 路由器
```javascript
mongos --configdb configReplSet/cfg1.example.net:27019,cfg2.example.net:27019,cfg3.example.net:27019 --port 27017
```

### 5. 添加分片到集群
```javascript
// 连接到mongos
mongo --host mongos.example.net --port 27017

// 添加分片
sh.addShard("shardReplA/shardA1.example.net:27018,shardA2.example.net:27018")
```

## 数据分区实战示例

### 创建分片集合
```javascript
// 1. 首先在admin数据库启用分片
sh.enableSharding("ecommerce")

// 2. 选择分片键(这里使用复合分片键)
sh.shardCollection("ecommerce.orders", { "customer_id": 1, "order_date": 1 })

// 3. 创建索引支持分片键
db.orders.createIndex({ "customer_id": 1, "order_date": 1 })
```

### 查看分片状态
```javascript
sh.status()
```

典型返回结果：
```
```json
{
  "shardingVersion": {
    "configVersion": 5,
    "minCompatibleVersion": 5,
    "currentVersion": 6
  },
  "shards": [
    {
      "_id": "shardReplA",
      "host": "shardReplA/shardA1.example.net:27018,shardA2.example.net:27018",
      "state": 1,
      "tags": []
    }
  ],
  "databases": [
    {
      "database": "ecommerce",
      "primary": "shardReplA",
      "partitioned": true,
      "collections": {
        "ecommerce.orders": {
          "shardKey": { "customer_id": 1, "order_date": 1 },
          "unique": false,
          "chunks": {
            "shardReplA": 2
          }
        }
      }
    }
  ]
}
```
```

## 分区策略深度对比

```
| 策略类型   | 数据分布方式                  | 优点                          | 缺点                          | 典型应用场景                 |
|------------|-------------------------------|-------------------------------|-------------------------------|------------------------------|
| 范围分片   | 按键值范围分布                | 范围查询高效                  | 可能导致数据分布不均          | 时间序列数据、数值范围查询   |
| 哈希分片   | 使用哈希函数均匀分布          | 数据分布均匀                  | 无法高效执行范围查询          | 高吞吐量写入场景            |
| 标签分片   | 基于自定义标签分布            | 支持地理位置分布等特殊需求    | 需要手动管理                  | 多数据中心部署              |
| 复合分片   | 组合多个字段作为分片键        | 平衡查询和写入需求            | 选择复杂度高                  | 复杂查询模式                |
```

## 分区维护操作

### 添加新分片
```javascript
sh.addShard("shardReplB/shardB1.example.net:27018,shardB2.example.net:27018")
```

### 查看数据分布
```javascript
db.orders.getShardDistribution()
```

返回示例：
```
```json
{
  "shard": "shardReplA",
  "numObjects": 1250000,
  "dataSize": 5368709120,
  "storageSize": 6442450944,
  "indexes": 2,
  "indexSize": 2147483648
}
```
```

### 手动迁移数据块
```javascript
sh.moveChunk("ecommerce.orders", { "customer_id": 500000 }, "shardReplB")
```

## 分区最佳实践

1. **分片键选择原则**：
   - 基数高（大量不同值）
   - 写分布均匀
   - 匹配查询模式

2. **避免热点**：不要使用单调递增的值作为唯一分片键

3. **预分配分片**：在数据增长前提前添加分片

4. **监控指标**：
   - 分片间的数据均衡情况
   - 各个分片的负载情况
   - 块迁移频率

5. **备份策略**：配置服务器必须包含在备份计划中

通过合理设计和维护分区策略，MongoDB 可以支持 PB 级数据存储和每秒数百万次操作的高性能数据库系统。