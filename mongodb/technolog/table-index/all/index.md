# MongoDB 索引类型详解

MongoDB 提供了多种索引类型来支持各种查询需求，下面我将详细介绍主要的索引类型及其使用示例。

## 1. 单字段索引 (Single Field Index)

最基本的索引类型，针对单个字段建立索引。

**示例：**
```javascript
// 创建单字段索引
db.users.createIndex({ name: 1 })

// 查询使用索引
db.users.find({ name: "张三" }).explain("executionStats")
```

**返回结果示例：**
```
{
  "queryPlanner": {
    "winningPlan": {
      "stage": "FETCH",
      "inputStage": {
        "stage": "IXSCAN",
        "keyPattern": { "name": 1 },
        "indexName": "name_1"
      }
    }
  },
  "executionStats": {
    "nReturned": 1,
    "executionTimeMillis": 0,
    "totalKeysExamined": 1,
    "totalDocsExamined": 1
  }
}
```

## 2. 复合索引 (Compound Index)

在多个字段上建立的索引，字段顺序很重要。

**示例：**
```javascript
// 创建复合索引
db.orders.createIndex({ customerId: 1, orderDate: -1 })

// 查询使用复合索引
db.orders.find({ 
  customerId: "12345", 
  orderDate: { $gte: ISODate("2023-01-01") } 
})
```

## 3. 多键索引 (Multikey Index)

用于数组字段的索引，为数组中的每个元素创建索引条目。

**示例：**
```javascript
// 创建多键索引
db.products.createIndex({ tags: 1 })

// 查询使用多键索引
db.products.find({ tags: "electronics" })
```

## 4. 文本索引 (Text Index)

支持全文搜索的特殊索引类型。

**示例：**
```javascript
// 创建文本索引
db.articles.createIndex({ content: "text" })

// 全文搜索
db.articles.find({ $text: { $search: "mongodb tutorial" } })
```

## 5. 地理空间索引 (Geospatial Index)

支持地理空间查询的索引，包括2dsphere和2d两种。

**示例：**
```javascript
// 创建2dsphere索引
db.places.createIndex({ location: "2dsphere" })

// 地理空间查询
db.places.find({
  location: {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [ -73.9667, 40.78 ]
      },
      $maxDistance: 1000
    }
  }
})
```

## 6. 哈希索引 (Hashed Index)

将字段值转换为哈希值的索引，主要用于分片键。

**示例：**
```javascript
// 创建哈希索引
db.users.createIndex({ username: "hashed" })

// 使用哈希索引查询
db.users.find({ username: "user123" })
```

## 7. [通配符索引 (Wildcard Index)](wildcard-index/index)

支持对未知或任意字段的查询。

**示例：**
```javascript
// 创建通配符索引
db.userData.createIndex({ "metadata.$**": 1 })

// 使用通配符索引查询
db.userData.find({ "metadata.age": { $gt: 30 } })
```

## 8. TTL索引 (TTL Index)

特殊索引，用于自动删除过期的文档。

**示例：**
```javascript
// 创建TTL索引（文档在创建后24小时自动删除）
db.logs.createIndex({ createdAt: 1 }, { expireAfterSeconds: 86400 })
```

## 索引类型对比表

```
| 索引类型       | 适用场景                          | 是否支持排序 | 是否支持数组 | 特殊功能               |
|----------------|-----------------------------------|--------------|--------------|------------------------|
| 单字段索引     | 单个字段查询                      | 是           | 否           | -                      |
| 复合索引       | 多字段组合查询                    | 是           | 否           | 字段顺序重要           |
| 多键索引       | 数组字段查询                      | 是           | 是           | 为每个数组元素建索引   |
| 文本索引       | 全文搜索                          | 否           | 否           | 支持文本评分           |
| 地理空间索引   | 地理位置查询                      | 否           | 否           | 支持地理空间操作       |
| 哈希索引       | 分片键                            | 否           | 否           | 均匀分布数据           |
| 通配符索引     | 不确定字段查询                    | 是           | 是           | 支持嵌套文档           |
| TTL索引        | 自动过期数据                      | 是           | 否           | 自动删除文档           |
```

## 索引使用建议

1. 为常用查询条件创建索引
2. 复合索引字段顺序遵循ESR原则(等值-排序-范围)
3. 避免在频繁更新的字段上创建过多索引
4. 使用`explain()`分析查询是否使用了索引
5. 定期监控索引使用情况，删除无用索引

**查看集合索引：**
```javascript
db.collection.getIndexes()
```

**删除索引：**
```javascript
db.collection.dropIndex("index_name")
```

希望这些信息能帮助您更好地理解和使用MongoDB的索引功能！