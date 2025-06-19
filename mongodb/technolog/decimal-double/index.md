# MongoDB 中的 Decimal 和 Double 数据类型介绍

在 MongoDB 中，`decimal` 和 `double` 都是用于存储数值的数据类型，但它们有着不同的特性和使用场景。

## 1. Decimal 类型

Decimal 类型（Decimal128）是 MongoDB 3.4 版本引入的高精度浮点数类型，主要用于需要精确计算的场景，如金融数据。

**特点：**
- 高精度，支持 34 位十进制数字
- 精确表示小数，不会出现浮点数精度问题
- 占用 16 字节存储空间

**示例：**

```javascript
// 插入包含 decimal 类型的数据
db.products.insertOne({
  name: "高端笔记本电脑",
  price: NumberDecimal("9999.99"),
  tax: NumberDecimal("0.15")
})

// 查询 decimal 数据
db.products.find({ name: "高端笔记本电脑" })
```

**返回结果：**
```
[
  {
    "_id": ObjectId("5f8d8a7b9d1e8c3a7c6b5a4d"),
    "name": "高端笔记本电脑",
    "price": NumberDecimal("9999.99"),
    "tax": NumberDecimal("0.15")
  }
]
```

## 2. Double 类型

Double 是标准的 IEEE 754 双精度浮点数类型，适用于大多数数值计算场景。

**特点：**
- 64位双精度浮点数
- 计算速度快
- 可能存在浮点数精度问题
- 占用 8 字节存储空间

**示例：**

```javascript
// 插入包含 double 类型的数据
db.products.insertOne({
  name: "普通笔记本电脑",
  price: 4999.99,
  tax: 0.15
})

// 查询 double 数据
db.products.find({ name: "普通笔记本电脑" })
```

**返回结果：**
```
[
  {
    "_id": ObjectId("5f8d8a7b9d1e8c3a7c6b5a4e"),
    "name": "普通笔记本电脑",
    "price": 4999.99,
    "tax": 0.15
  }
]
```

## 3. Decimal 与 Double 对比

```
| 特性                | Decimal                      | Double                      |
|---------------------|------------------------------|-----------------------------|
| 精度                | 34位十进制数字               | 约15-17位十进制数字         |
| 存储空间            | 16字节                       | 8字节                       |
| 计算速度            | 较慢                         | 较快                        |
| 适用场景            | 金融、精确计算               | 一般数值计算                |
| 精度问题            | 无                           | 可能存在                    |
| MongoDB 版本要求    | 3.4+                         | 所有版本                    |
| 表示方式            | NumberDecimal("123.45")      | 123.45                      |
```

## 4. 实际使用建议

1. **金融数据**：如价格、税率、货币金额等，使用 Decimal 类型
2. **科学计算**：如测量数据、统计数据等，使用 Double 类型
3. **整数**：可以使用 Integer 或 Long 类型

**精度问题示例：**

```javascript
// 使用 double 计算可能出现的精度问题
db.calculations.insertOne({ a: 0.1, b: 0.2 })
db.calculations.aggregate([
  { $project: { sum: { $add: ["$a", "$b"] } } }
])
```

**返回结果：**
```
[
  {
    "_id": ObjectId("5f8d8a7b9d1e8c3a7c6b5a4f"),
    "sum": 0.30000000000000004
  }
]
```

```javascript
// 使用 decimal 计算
db.calculations.insertOne({ a: NumberDecimal("0.1"), b: NumberDecimal("0.2") })
db.calculations.aggregate([
  { $project: { sum: { $add: ["$a", "$b"] } } }
])
```

**返回结果：**
```
[
  {
    "_id": ObjectId("5f8d8a7b9d1e8c3a7c6b5a50"),
    "sum": NumberDecimal("0.3")
  }
]
```

## 5. 查询和索引

Decimal 和 Double 都可以创建索引，但在查询时需要注意类型匹配：

```javascript
// 创建索引
db.products.createIndex({ price: 1 })

// 查询 double 值
db.products.find({ price: 4999.99 })

// 查询 decimal 值 - 必须使用 NumberDecimal
db.products.find({ price: NumberDecimal("9999.99") })
```

希望这些信息能帮助您理解 MongoDB 中的 Decimal 和 Double 类型！