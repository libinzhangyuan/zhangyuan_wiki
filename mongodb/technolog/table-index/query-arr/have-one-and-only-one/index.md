# MongoDB 数组精确匹配单个元素

您的问题是关于如何查询数组中**只包含**特定单个元素的文档。`db.students.find({ scores: 80 })` 这种写法只能匹配数组中**包含**80的文档，但不能确保数组中**只有**80这一个元素。

## 几种不同查询方式的区别

假设有以下数据：

```javascript
db.students.insertMany([
  { name: "张三", scores: [80] },             // 只有80
  { name: "李四", scores: [80, 90] },         // 包含80和其他元素
  { name: "王五", scores: [70, 80, 85] },      // 包含80和其他元素
  { name: "赵六", scores: [90] }              // 不包含80
])
```

### 1. 基本包含查询（您提到的方式）

```javascript
db.students.find({ scores: 80 })
```

返回结果：
```
[
  { "_id": ObjectId("..."), "name": "张三", "scores": [80] },
  { "_id": ObjectId("..."), "name": "李四", "scores": [80, 90] },
  { "_id": ObjectId("..."), "name": "王五", "scores": [70, 80, 85] }
]
```

这种方式会返回**所有包含80**的文档，不论数组中有多少其他元素。

### 2. 精确匹配单个元素（您想要的查询）

要查询数组中**只有**80一个元素的文档，需要使用`$size`和`$eq`操作符组合：

```javascript
db.students.find({
  scores: {
    $eq: [80]  // 数组完全等于[80]
  }
})
```

或者更灵活的方式：

```javascript
db.students.find({
  scores: {
    $all: [80],    // 包含80
    $size: 1       // 数组长度为1
  }
})
```

返回结果：
```
[
  { "_id": ObjectId("..."), "name": "张三", "scores": [80] }
]
```

## 对比表

```
| 查询方式                      | 匹配条件                     | 返回文档           |
|-------------------------------|-----------------------------|--------------------|
| {scores: 80}                 | 数组中包含80                | 张三,李四,王五     |
| {scores: [80]}               | 数组完全等于[80]            | 张三               |
| {scores: {$all: [80], $size: 1}} | 包含80且数组长度为1 | 张三               |
```

## 其他相关查询示例

### 1. 查询数组长度为2且包含80的文档

```javascript
db.students.find({
  scores: {
    $all: [80],
    $size: 2
  }
})
```

返回结果：
```
[
  { "_id": ObjectId("..."), "name": "李四", "scores": [80, 90] }
]
```

### 2. 使用聚合框架实现复杂匹配

```javascript
db.students.aggregate([
  {
    $match: {
      scores: 80
    }
  },
  {
    $addFields: {
      scoresCount: { $size: "$scores" }
    }
  },
  {
    $match: {
      scoresCount: 1
    }
  }
])
```

## 总结

1. `{ field: value }` - 匹配数组中**包含**该值的文档
2. `{ field: [value] }` - 匹配数组**完全等于**[value]的文档（严格匹配）
3. `{ field: { $all: [value], $size: 1 } }` - 匹配**包含**该值且**数组长度为1**的文档（最符合您需求的方式）


{ field: value } 是最基础的精确匹配

{ field: [value1, value2] } 要求数组完全匹配

$all 操作符不关心元素顺序，只要求包含所有指定元素

$size 操作符可以限制数组长度
{ field: { $all: [value1, value2], $size: 2 }} 不关系顺序。


组合使用 $all 和 $size 可以实现更精确的数组查询

