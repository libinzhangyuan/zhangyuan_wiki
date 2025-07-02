# MongoDB 稀疏索引介绍：ensureIndex({sparse: true})

在 MongoDB 中，稀疏索引(`sparse index`)是一种特殊类型的索引，它只包含那些具有索引字段的文档条目（即使字段值为null也会包含，但不包含没有该字段的文档）。

## 稀疏索引的基本语法

```javascript
db.collection.ensureIndex({ field: 1 }, { sparse: true })
```

## 稀疏索引的特点

1. 只索引包含索引字段的文档（即使字段值为null）
2. 不索引完全缺少该字段的文档
3. 可以节省存储空间和索引维护开销
4. 对于唯一稀疏索引，允许多个文档缺少该字段（不视为重复）

## 示例

### 1. 创建普通稀疏索引

```javascript
// 在users集合的middleName字段上创建稀疏索引
// 只有包含middleName字段的文档会被索引
db.users.ensureIndex({ middleName: 1 }, { sparse: true })
```

返回结果示例：
```
{
  "createdCollectionAutomatically" : false,
  "numIndexesBefore" : 1,
  "numIndexesAfter" : 2,
  "ok" : 1
}
```

### 2. 创建唯一稀疏索引

```javascript
// 在products集合的serialNumber字段上创建唯一稀疏索引
// 允许没有serialNumber字段的文档，但有该字段的值必须唯一
db.products.ensureIndex({ serialNumber: 1 }, { unique: true, sparse: true })
```

返回结果示例：
```
{
  "createdCollectionAutomatically" : false,
  "numIndexesBefore" : 2,
  "numIndexesAfter" : 3,
  "ok" : 1
}
```

## 稀疏索引查询行为

### 数据准备
```javascript
db.users.insertMany([
  { _id: 1, name: "Alice", middleName: "Anne" },
  { _id: 2, name: "Bob" }, // 没有middleName字段
  { _id: 3, name: "Charlie", middleName: null }
])
```

### 查询示例1：使用稀疏索引字段查询

```javascript
// 使用稀疏索引字段查询
db.users.find({ middleName: { $exists: true } }).explain("executionStats")
```

返回结果（部分）：
```
{
  "queryPlanner" : {
    "winningPlan" : {
      "stage" : "FETCH",
      "inputStage" : {
        "stage" : "IXSCAN",  // 使用了索引扫描
        "indexName" : "middleName_1"
      }
    }
  }
}
```

### 查询示例2：查询不存在的字段

```javascript
// 查询没有middleName字段的文档
db.users.find({ middleName: { $exists: false } }).explain("executionStats")
```

返回结果（部分）：
```
{
  "queryPlanner" : {
    "winningPlan" : {
      "stage" : "COLLSCAN",  // 需要全表扫描，无法使用稀疏索引
    }
  }
}
```

## 稀疏索引 vs 普通索引

```
| 特性                | 稀疏索引                     | 普通索引                     |
|---------------------|----------------------------|----------------------------|
| 索引范围             | 只索引包含该字段的文档         | 索引所有文档                 |
| 存储空间             | 通常更小                    | 通常更大                    |
| 查询性能             | 对包含字段的查询更快           | 对所有文档查询一致            |
| 唯一性约束           | 允许缺少字段的文档            | 所有文档必须满足唯一性         |
| 适用场景             | 字段非必填且查询频率不高        | 字段必填或查询频率高           |
```

## 使用建议

1. 适合用于可选字段的索引
2. 当字段在大多数文档中不存在时特别有用
3. 结合唯一约束时，可以实现"存在则唯一"的业务逻辑
4. 注意查询行为的变化，特别是使用`$exists`操作符时

## 查看稀疏索引

```javascript
db.users.getIndexes()
```

返回结果示例：
```
[
  {
    "v" : 2,
    "key" : {
      "_id" : 1
    },
    "name" : "_id_",
    "ns" : "test.users"
  },
  {
    "v" : 2,
    "key" : {
      "middleName" : 1
    },
    "name" : "middleName_1",
    "ns" : "test.users",
    "sparse" : true
  }
]
```