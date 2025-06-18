[返回](/mongodb/technolog/index)


# MongoDB 统计数量慢问题分析

## 为什么统计数量会慢？

在MongoDB中，统计文档数量(`count`)操作可能会变慢的主要原因包括：

1. **集合大小**：当集合中文档数量非常大时(百万级以上)，统计操作会变慢
2. **缺少索引**：如果没有合适的索引支持查询条件，MongoDB需要扫描整个集合
3. **分片集群**：在分片环境中，count操作需要从所有分片获取数据并汇总
4. **过滤条件复杂**：带有复杂查询条件的count会比简单count慢很多

## 基本count命令示例

```javascript
// 统计集合中所有文档数量
db.orders.count()

// 返回结果示例
```
```json
1254783
```

```javascript
// 带条件的统计
db.orders.count({ status: "shipped" })

// 返回结果示例
```
```json
452189
```

## 性能优化方案

### 1. 使用estimatedDocumentCount()获取近似值

当不需要精确计数时，可以使用这个方法，它直接从集合元数据获取计数，速度非常快。

```javascript
db.orders.estimatedDocumentCount()

// 返回结果示例
```
```json
1254000
```

### 2. 为查询条件创建索引

```javascript
// 创建索引
db.orders.createIndex({ status: 1 })

// 然后执行count
db.orders.count({ status: "shipped" })
```

### 3. 使用countDocuments()替代count()

`count()`在MongoDB 4.0+已被标记为废弃，推荐使用`countDocuments()`

```javascript
db.orders.countDocuments({ status: "shipped" })

// 返回结果示例
```
```json
452189
```

### 4. 分页场景下的优化方案

```javascript
// 不好的做法 - 先count再find
let total = db.orders.countDocuments({ status: "shipped" })
let data = db.orders.find({ status: "shipped" }).skip(0).limit(10)

// 好的做法 - 只获取数据，前端展示"更多"而不是总数量
let data = db.orders.find({ status: "shipped" }).skip(0).limit(10)
```

## 性能对比

以下是在100万文档集合上的性能对比：

| 方法 | 执行时间(ms) | 精确性 | 适用场景 |
|------|-------------|--------|----------|
| `count()` | 1200 | 精确 | 需要精确计数的小集合 |
| `estimatedDocumentCount()` | 2 | 近似 | 展示大致数量 |
| `countDocuments()`有索引 | 15 | 精确 | 带条件的精确计数 |
| `countDocuments()`无索引 | 980 | 精确 | 不推荐 |

## 监控慢查询

```javascript
// 查看慢查询日志
db.setProfilingLevel(1, { slowms: 50 })

// 查看分析结果
db.system.profile.find().sort({ ts: -1 }).limit(10)

// 返回结果示例
```
```json
[
  {
    "op": "command",
    "ns": "test.orders",
    "command": {
      "count": "orders",
      "query": { "status": "shipped" }
    },
    "millis": 1245,
    "ts": ISODate("2023-05-20T03:23:43.234Z")
  }
]
```

## 总结

对于大型集合，尽量避免频繁执行精确的count操作。根据业务需求选择：
- 需要精确值且数据量小 → `countDocuments()`
- 需要近似值 → `estimatedDocumentCount()`
- 分页场景 → 考虑无限滚动或"加载更多"设计

记得为常用查询条件创建索引，这能显著提高count操作的性能。