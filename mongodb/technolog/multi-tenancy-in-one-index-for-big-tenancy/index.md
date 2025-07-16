[返回](/mongodb/technolog/index)

# MongoDB 为大租户创建专用索引

在 MongoDB 中为大租户(重要客户或数据量大的租户)创建专用索引是一种常见的多租户数据库优化策略，这可以显著提高大租户的查询性能。

## 为什么需要为大租户创建专用索引

1. 大租户通常数据量更大，查询需求更复杂
2. 避免小租户的查询影响大租户的性能
3. 可以针对大租户的特殊查询模式优化索引

## 创建专用索引的方法

### 1. 使用租户ID作为复合索引的一部分

```javascript
// 为orders集合创建包含tenant_id的复合索引
db.orders.createIndex({ tenant_id: 1, order_date: -1 })

// 查询示例
db.orders.find({
  tenant_id: "big_tenant_A",
  order_date: { $gte: ISODate("2023-01-01") }
}).explain("executionStats")
```

返回结果示例：
```
{
  "queryPlanner": {
    "winningPlan": {
      "stage": "FETCH",
      "inputStage": {
        "stage": "IXSCAN",
        "keyPattern": { "tenant_id": 1, "order_date": -1 },
        "indexName": "tenant_id_1_order_date_-1"
      }
    }
  },
  "executionStats": {
    "nReturned": 1250,
    "executionTimeMillis": 15,
    "totalKeysExamined": 1250,
    "totalDocsExamined": 1250
  }
}
```

### 2. 为特定租户创建独立索引

```javascript
// 只为大租户big_tenant_B创建特殊索引
db.customers.createIndex(
  { "tenant": 1, "loyalty_score": -1 },
  { name: "big_tenant_B_loyalty_idx", 
    partialFilterExpression: { tenant: "big_tenant_B" }
  }
)

// 查询大租户的高价值客户
db.customers.find({
  tenant: "big_tenant_B",
  loyalty_score: { $gt: 80 }
})
```

返回结果示例：
```
[
  {
    "_id": ObjectId("5f3d7e9c8e3a6d2a1c8b4567"),
    "tenant": "big_tenant_B",
    "name": "Premium Customer 1",
    "loyalty_score": 95
  },
  {
    "_id": ObjectId("5f3d7e9c8e3a6d2a1c8b4568"),
    "tenant": "big_tenant_B",
    "name": "Premium Customer 2",
    "loyalty_score": 88
  }
]
```

### 3. 使用分片集群为不同租户分配不同分片

```javascript
// 启用分片
sh.enableSharding("multi_tenant_db")

// 基于租户ID进行分片
sh.shardCollection("multi_tenant_db.products", { "tenant_id": 1 })

// 为大租户配置专属分片
sh.addShardTag("shard2", "big_tenant_A")
sh.addTagRange(
  "multi_tenant_db.products",
  { "tenant_id": "big_tenant_A" },
  { "tenant_id": "big_tenant_A" + "\uFFFF" },
  "big_tenant_A"
)
```

## 索引效果对比

### 查询性能对比
```
| 查询类型 | 无专用索引(ms) | 有专用索引(ms) | 提升幅度 |
|---------|--------------|--------------|---------|
| 大租户订单查询 | 450 | 15 | 30倍 |
| 大租户客户查询 | 320 | 8 | 40倍 |
| 混合租户查询 | 180 | 175 | 基本持平 |
```
### 资源使用对比

```
无专用索引的资源使用:
- 内存: 8GB
- CPU平均使用率: 65%
- 磁盘IOPS: 1200

有专用索引的资源使用:
- 内存: 9.5GB (增加了索引内存)
- CPU平均使用率: 35%
- 磁盘IOPS: 400
```

## 最佳实践建议

1. **评估租户数据量**：只为数据量超过一定阈值(如100万文档)的租户创建专用索引
2. **监控索引使用**：定期检查索引使用情况，删除未使用的索引
3. **考虑写入影响**：专用索引会增加写入开销，需平衡读写性能
4. **使用部分索引**：结合`partialFilterExpression`减少索引大小
5. **定期维护**：对大租户的索引定期进行重建(`reIndex`)

通过为大租户创建专用索引，可以在多租户环境中实现更均衡的性能分配，确保大租户的查询性能不会受到小租户的影响。