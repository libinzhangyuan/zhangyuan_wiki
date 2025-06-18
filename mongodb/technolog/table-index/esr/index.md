# MongoDB 索引设计中的 ESR 原则

ESR 原则(等值-排序-范围)是 MongoDB 索引设计的重要准则，它指导我们如何合理安排复合索引中字段的顺序，以最大化查询性能。

## ESR 原则详解

ESR 代表：
- **E**quality (等值查询) - 字段使用精确匹配(=, $eq)
- **S**ort (排序) - 字段用于排序操作
- **R**ange (范围查询) - 字段使用范围查询($gt, $lt, $in 等)

### 最佳顺序
复合索引中的字段应按以下顺序排列：
1. 等值查询字段
2. 排序字段
3. 范围查询字段

## 实际应用示例

### 示例集合结构
```javascript
{
  _id: ObjectId,
  userId: String,
  createDate: Date,
  status: String,
  price: Number,
  category: String
}
```

### 示例1：等值 + 排序
```javascript
// 查询：查找特定用户且按日期排序
db.orders.find({ userId: "user123" }).sort({ createDate: -1 })

// 最佳索引
db.orders.createIndex({ userId: 1, createDate: -1 })
```
```
索引字段顺序解释：
1. userId (等值查询)
2. createDate (排序)
```

### 示例2：等值 + 范围 + 排序
```javascript
// 查询：查找特定状态、价格范围内，按日期排序
db.orders.find({ 
  status: "completed", 
  price: { $gt: 100, $lt: 500 } 
}).sort({ createDate: 1 })

// 最佳索引
db.orders.createIndex({ status: 1, createDate: 1, price: 1 })
```
```
索引字段顺序解释：
1. status (等值查询)
2. createDate (排序)
3. price (范围查询)
```

### 示例3：多个等值 + 排序
```javascript
// 查询：查找特定用户和类别，按日期排序
db.orders.find({ 
  userId: "user123",
  category: "electronics"
}).sort({ createDate: -1 })

// 最佳索引
db.orders.createIndex({ userId: 1, category: 1, createDate: -1 })
```
```
索引字段顺序解释：
1. userId (等值查询)
2. category (等值查询)
3. createDate (排序)
```

## 错误设计对比

### 错误示例1：范围查询在排序前
```javascript
// 查询：查找价格大于100且按日期排序
db.orders.find({ price: { $gt: 100 } }).sort({ createDate: -1 })

// 错误索引
db.orders.createIndex({ price: 1, createDate: -1 })
```
```
问题分析：
范围查询字段(price)放在了排序字段(createDate)前面，导致排序无法有效使用索引
```

### 错误示例2：忽略等值查询
```javascript
// 查询：查找特定状态且价格大于100
db.orders.find({ 
  status: "completed",
  price: { $gt: 100 }
})

// 次优索引
db.orders.createIndex({ price: 1, status: 1 })
```
```
问题分析：
等值查询字段(status)应该放在范围查询字段(price)前面
```

## 复合索引使用对比

我们创建一个包含100万文档的orders集合，比较不同索引设计对查询性能的影响：

```javascript
// 创建测试数据
for (let i = 0; i < 1000000; i++) {
  db.orders.insert({
    userId: "user" + (i % 1000),
    createDate: new Date(2023, i % 12, i % 28),
    status: ["pending", "completed", "cancelled"][i % 3],
    price: Math.random() * 1000,
    category: ["electronics", "clothing", "food"][i % 3]
  })
}
```

### 查询性能对比

| 查询 | 索引设计 | 执行时间(ms) | 扫描文档数 |
|------|----------|-------------|------------|
| `find({userId:"user123"}).sort({createDate:-1})` | `{userId:1, createDate:-1}` | 2 | 1000 |
| `find({userId:"user123"}).sort({createDate:-1})` | `{createDate:-1, userId:1}` | 45 | 1000000 |
| `find({status:"completed", price:{$gt:100}})` | `{status:1, price:1}` | 5 | 330000 |
| `find({status:"completed", price:{$gt:100}})` | `{price:1, status:1}` | 25 | 500000 |

## 特殊情况处理

### 1. 多个范围查询
当查询中有多个范围条件时，ESR原则可能难以完全适用，此时应优先考虑最严格的范围条件。

```javascript
// 查询：查找特定用户，价格大于100且小于500，日期在特定范围内
db.orders.find({
  userId: "user123",
  price: { $gt: 100, $lt: 500 },
  createDate: { $gt: ISODate("2023-01-01"), $lt: ISODate("2023-06-01") }
})

// 推荐索引
db.orders.createIndex({ userId: 1, createDate: 1, price: 1 })
```

### 2. 多字段排序
当有多个排序字段时，应将它们按顺序放在索引中。

```javascript
// 查询：查找特定状态，按类别升序和日期降序排序
db.orders.find({ status: "completed" })
         .sort({ category: 1, createDate: -1 })

// 最佳索引
db.orders.createIndex({ status: 1, category: 1, createDate: -1 })
```

## 总结

ESR原则是MongoDB索引设计的黄金法则，合理应用可以显著提高查询性能。实际应用中应注意：
1. 精确匹配字段放在索引最前面
2. 排序字段放在中间位置
3. 范围查询字段放在最后
4. 避免将范围查询字段放在排序字段前面
5. 使用`explain()`分析查询执行计划，验证索引效果