# MongoDB 数组匹配查询详解

## 基本数组查询

### 1. 精确匹配整个数组(元素和顺序完全一致)

```javascript
// 数据准备
db.students.insertMany([
  { name: "张三", scores: [80, 85, 90] },
  { name: "李四", scores: [75, 80, 85] },
  { name: "王五", scores: [90, 95, 100] }
])

// 查询scores数组完全等于[80,85,90]的文档
db.students.find({ scores: [80, 85, 90] })
```

返回结果：
```
[
  { "_id": ObjectId("..."), "name": "张三", "scores": [80, 85, 90] }
]
```

### 2. 匹配数组中的单个元素

```javascript
// 查询scores数组中包含80的文档
db.students.find({ scores: 80 })
```

返回结果：
```
[
  { "_id": ObjectId("..."), "name": "张三", "scores": [80, 85, 90] },
  { "_id": ObjectId("..."), "name": "李四", "scores": [75, 80, 85] }
]
```

## 高级数组查询操作符

### 1. $all - 匹配包含所有指定元素的数组

```javascript
// 查询scores数组中同时包含80和85的文档
db.students.find({ scores: { $all: [80, 85] } })
```

返回结果：
```
[
  { "_id": ObjectId("..."), "name": "张三", "scores": [80, 85, 90] },
  { "_id": ObjectId("..."), "name": "李四", "scores": [75, 80, 85] }
]
```

### 2. $size - 按数组长度查询

```javascript
// 查询scores数组长度为3的文档
db.students.find({ scores: { $size: 3 } })
```

返回结果：
```
[
  { "_id": ObjectId("..."), "name": "张三", "scores": [80, 85, 90] },
  { "_id": ObjectId("..."), "name": "李四", "scores": [75, 80, 85] },
  { "_id": ObjectId("..."), "name": "王五", "scores": [90, 95, 100] }
]
```

### 3. $elemMatch - 匹配满足多个条件的数组元素

```javascript
// 数据准备
db.quizzes.insertMany([
  { results: [ { product: "A", score: 8 }, { product: "B", score: 7 } ] },
  { results: [ { product: "A", score: 7 }, { product: "B", score: 8 } ] }
])

// 查询results数组中至少有一个元素同时满足product为"A"且score大于7的文档
db.quizzes.find({ 
  results: { 
    $elemMatch: { 
      product: "A", 
      score: { $gt: 7 } 
    } 
  } 
})
```

返回结果：
```
[
  { 
    "_id": ObjectId("..."), 
    "results": [ 
      { "product": "A", "score": 8 }, 
      { "product": "B", "score": 7 } 
    ] 
  }
]
```

## 数组索引查询

### 1. 多键索引对数组查询的影响

```javascript
// 创建多键索引
db.students.createIndex({ scores: 1 })

// 使用索引查询
db.students.find({ scores: 80 }).explain("executionStats")
```

返回结果（部分）：
```
{
  "queryPlanner": {
    "winningPlan": {
      "stage": "FETCH",
      "inputStage": {
        "stage": "IXSCAN",  // 使用了索引扫描
        "keyPattern": { "scores": 1 },
        "indexName": "scores_1",
        ...
      }
    }
  },
  "executionStats": {
    "nReturned": 2,
    "executionTimeMillis": 0,
    "totalKeysExamined": 2,
    ...
  }
}
```

## 数组位置查询

### 1. 使用点表示法查询特定位置的元素

```javascript
// 查询scores数组的第一个元素等于80的文档
db.students.find({ "scores.0": 80 })
```

返回结果：
```
[
  { "_id": ObjectId("..."), "name": "张三", "scores": [80, 85, 90] }
]
```

### 2. $ 操作符 - 匹配并返回符合条件的数组元素

```javascript
// 数据准备
db.students.updateOne(
  { name: "张三" },
  { $push: { comments: { text: "优秀", by: "老师" } } }
)

// 查询并返回匹配的数组元素
db.students.find(
  { "comments.by": "老师" },
  { "comments.$": 1 }
)
```

返回结果：
```
[
  {
    "_id": ObjectId("..."),
    "comments": [
      { "text": "优秀", "by": "老师" }
    ]
  }
]
```

## 数组查询性能对比

```
| 查询方式               | 索引支持 | 性能 | 适用场景                     |
|-----------------------|---------|------|----------------------------|
| 精确匹配整个数组       | 否      | 低   | 元素和顺序完全一致            |
| 单个元素匹配           | 是      | 高   | 查找包含某元素的文档        |
| $all 操作符           | 部分    | 中   | 查找包含多个元素的文档      |
| $elemMatch            | 是      | 中高 | 数组元素需满足多个条件      |
| 位置查询(如array.0)   | 是      | 高   | 查询特定位置的元素          |
| $size                 | 否      | 低   | 按数组长度查询              |
```

## 最佳实践建议

1. 为经常查询的数组字段创建多键索引
2. 避免使用$size操作符查询大数组
3. 使用$elemMatch代替多个独立条件查询
4. 对于复杂的数组查询，考虑使用聚合管道
5. 合理设计数据结构，避免过大的数组

通过合理使用这些数组查询技巧，可以高效地从MongoDB中检索包含数组字段的文档。