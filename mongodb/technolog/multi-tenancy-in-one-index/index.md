[返回](/mongodb/technolog/index)

# MongoDB 文档级别多租户的索引设计

在文档级别多租户设计中，所有租户的数据存储在同一个集合中，通过`tenant_id`字段区分。这种设计下的索引策略需要特别注意，既要保证查询性能，又要避免索引膨胀。

## 基础索引设计

### 1. 必须的租户ID索引

首先，所有查询都应该包含`tenant_id`条件，因此必须为`tenant_id`创建索引。

```javascript
// 创建基本的租户ID索引
db.products.createIndex({tenant_id: 1})

// 复合索引示例(tenant_id + 其他字段)
db.products.createIndex({tenant_id: 1, name: 1})
```

查询示例：
```javascript
db.products.find({
  tenant_id: "A",
  name: "Product A2"
}).explain("executionStats")
```

返回结果：
```
{
  "queryPlanner": {
    "winningPlan": {
      "stage": "FETCH",
      "inputStage": {
        "stage": "IXSCAN",
        "keyPattern": {"tenant_id": 1, "name": 1},
        "indexName": "tenant_id_1_name_1"
      }
    }
  }
}
```

### 2. 租户ID索引设计模式

```
| 索引类型                | 示例                               | 适用场景                          |
|-------------------------|------------------------------------|-----------------------------------|
| 单字段索引              | {tenant_id: 1}                     | 仅按租户过滤的查询                |
| 复合索引(租户前置)       | {tenant_id: 1, name: 1}            | 按租户和其他字段过滤的查询         |
| 复合索引(租户+排序字段)  | {tenant_id: 1, price: -1}          | 按租户过滤并需要排序的查询         |
| 多键索引(数组字段)       | {tenant_id: 1, tags: 1}            | 租户数据中包含数组字段的查询       |
```

## 高级索引策略

### 1. 部分索引(Partial Index)

只为特定租户或满足条件的文档创建索引，减少索引大小。

```javascript
// 只为VIP租户创建索引
db.products.createIndex(
  {name: 1},
  {partialFilterExpression: {tenant_id: "VIP"}}
)

// 查询会使用此部分索引
db.products.find({
  tenant_id: "VIP",
  name: "Premium Product"
})
```

### 2. 稀疏索引(Sparse Index)

只为包含该字段的文档创建索引，适合可选字段。

```javascript
// 只为有discount字段的文档创建索引
db.products.createIndex(
  {discount: 1},
  {sparse: true}
)
```

### 3. TTL索引与租户结合

可以按租户设置不同的数据过期时间。

```javascript
// 租户A的数据7天后过期，租户B的数据30天后过期
db.logs.createIndex(
  {createdAt: 1},
  {
    partialFilterExpression: {tenant_id: "A"},
    expireAfterSeconds: 604800 // 7天
  }
)

db.logs.createIndex(
  {createdAt: 1},
  {
    partialFilterExpression: {tenant_id: "B"},
    expireAfterSeconds: 2592000 // 30天
  }
)
```

## 分片集群中的索引设计

在分片环境中，索引设计更加关键：

```javascript
// 分片键通常应包含tenant_id
sh.shardCollection("mydb.products", {tenant_id: 1, _id: 1})

// 然后在分片键上创建索引
db.products.createIndex({tenant_id: 1, _id: 1})
```

## 索引优化实践

### 1. 索引选择性测试

```javascript
// 检查索引选择性
db.products.aggregate([
  {$match: {tenant_id: "A"}},
  {$group: {_id: "$name", count: {$sum: 1}}},
  {$sort: {count: -1}},
  {$limit: 5}
])
```

示例结果：
```
[
  {"_id": "Common Product", "count": 1250},
  {"_id": "Standard Item", "count": 876},
  {"_id": "Basic Model", "count": 543},
  {"_id": "Premium Offer", "count": 210},
  {"_id": "Special Edition", "count": 98}
]
```

### 2. 索引性能对比

```
| 查询类型                 | 无索引性能 | 正确索引性能 | 索引大小 |
|--------------------------|------------|--------------|----------|
| 单租户基础查询           | 120ms      | 2ms          | 小型     |
| 单租户+字段过滤          | 95ms       | 3ms          | 中型     |
| 单租户+排序              | 210ms      | 5ms          | 中型     |
| 跨租户查询(管理员)        | 45ms       | 40ms         | 大型     |
```

### 3. 索引维护建议

1. 定期监控索引使用情况：
```javascript
db.products.aggregate([{$indexStats: {}}])
```

2. 移除未使用的索引：
```javascript
db.products.dropIndex("name_1")
```

3. 在低峰期创建大型索引：
```javascript
db.products.createIndex(
  {tenant_id: 1, description: "text"},
  {background: true}
)
```

## 常见问题解决方案

### 1. 热点租户问题

对于数据量特别大的租户，考虑单独处理：

```javascript
// 为大租户创建专用索引
db.products.createIndex(
  {tenant_id: 1, category: 1},
  {partialFilterExpression: {tenant_id: "LargeTenant"}}
)
```

### 2. 跨租户查询优化

管理员查询需要访问所有租户数据：

```javascript
// 为管理员查询创建不含tenant_id的索引
db.products.createIndex({name: 1})

// 或者使用特殊的all_tenants标志
db.products.createIndex(
  {all_tenants: 1, name: 1},
  {partialFilterExpression: {all_tenants: true}}
)
```

### 3. 多租户索引内存优化

```javascript
// 使用collation为不同租户设置不同排序规则
db.products.createIndex(
  {name: 1},
  {collation: {locale: "en", strength: 2}}
)

// 限制索引内存使用
db.adminCommand({
  setParameter: 1,
  maxIndexBuildMemoryUsageMegabytes: 1024
})
```

文档级别多租户的索引设计核心原则是：**所有索引都应该以tenant_id开头**，除非是特别为管理员查询设计的索引。同时要定期审查索引使用情况，移除冗余索引，确保系统性能最优。