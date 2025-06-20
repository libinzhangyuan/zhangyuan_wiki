# MongoDB 中的 Number 类型 (BSON)

在 MongoDB 中，数字类型是 BSON (Binary JSON) 格式的一部分，用于存储数值数据。MongoDB 支持多种数字类型，每种类型有不同的特性和使用场景。

## MongoDB 中的数字类型

MongoDB 主要支持以下数字类型：

1. **32位整数 (Int32)** - 存储 32 位有符号整数
2. **64位整数 (Int64)** - 存储 64 位有符号整数
3. **双精度浮点数 (Double)** - 存储 64 位 IEEE 754 浮点数
4. **高精度小数 (Decimal128)** - 存储 128 位 IEEE 754-2008 十进制浮点数

## 数字类型对比

```
| 类型        | 存储大小 | 范围                                      | 精度          | 使用场景                     |
|-------------|----------|------------------------------------------|---------------|----------------------------|
| Int32       | 4字节    | -2,147,483,648 到 2,147,483,647         | 精确整数       | 计数器、年龄等小整数         |
| Int64       | 8字节    | -9,223,372,036,854,775,808 到 9,223,372,036,854,775,807 | 精确整数 | 大整数ID、时间戳等          |
| Double      | 8字节    | 约 ±1.7e308 (15-17位有效数字)            | 近似浮点数     | 科学计算、地理坐标等         |
| Decimal128  | 16字节   | ±1.0e-6143 到 ±1.0e+6144                 | 精确小数       | 金融数据、需要精确计算的数值 |
```

## 示例操作

### 1. 插入不同数字类型的文档

```javascript
db.products.insertMany([
  { name: "商品A", price: 25.99, quantity: 100, rating: 4.5, taxRate: NumberDecimal("0.0825") },
  { name: "商品B", price: 1299.99, quantity: 5, rating: 4.8, taxRate: NumberDecimal("0.0925") },
  { name: "商品C", price: 9.99, quantity: 500, rating: 3.9, taxRate: NumberDecimal("0.0625") }
])
```

插入后查询结果：

```
[
  {
    "_id": ObjectId("5f8d8a7b4f4d4b4d4b4d4b4b"),
    "name": "商品A",
    "price": 25.99,
    "quantity": 100,
    "rating": 4.5,
    "taxRate": NumberDecimal("0.0825")
  },
  {
    "_id": ObjectId("5f8d8a7b4f4d4b4d4b4d4b4c"),
    "name": "商品B",
    "price": 1299.99,
    "quantity": 5,
    "rating": 4.8,
    "taxRate": NumberDecimal("0.0925")
  },
  {
    "_id": ObjectId("5f8d8a7b4f4d4b4d4b4d4b4d"),
    "name": "商品C",
    "price": 9.99,
    "quantity": 500,
    "rating": 3.9,
    "taxRate": NumberDecimal("0.0625")
  }
]
```

### 2. 查询中使用数字比较

```javascript
// 查询价格大于100的商品
db.products.find({ price: { $gt: 100 } })
```

返回结果：

```
[
  {
    "_id": ObjectId("5f8d8a7b4f4d4b4d4b4d4b4c"),
    "name": "商品B",
    "price": 1299.99,
    "quantity": 5,
    "rating": 4.8,
    "taxRate": NumberDecimal("0.0925")
  }
]
```

### 3. 使用聚合计算

```javascript
// 计算所有商品的总价值 (price * quantity)
db.products.aggregate([
  {
    $project: {
      name: 1,
      totalValue: { $multiply: ["$price", "$quantity"] }
    }
  }
])
```

返回结果：

```
[
  {
    "_id": ObjectId("5f8d8a7b4f4d4b4d4b4d4b4b"),
    "name": "商品A",
    "totalValue": 2599
  },
  {
    "_id": ObjectId("5f8d8a7b4f4d4b4d4b4d4b4c"),
    "name": "商品B",
    "totalValue": 6499.95
  },
  {
    "_id": ObjectId("5f8d8a7b4f4d4b4d4b4d4b4d"),
    "name": "商品C",
    "totalValue": 4995
  }
]
```

## 注意事项

1. **类型转换**：MongoDB 会自动转换一些数字类型，这可能导致精度问题
   ```javascript
   // 插入一个整数
   db.test.insertOne({ value: 5 }) # 这个实际存储的是double 5.0
   
   // 查询时会匹配不同类型的5
   db.test.find({ value: 5.0 }) // 会匹配到上面的文档
   ```

2. **Decimal128 的特殊性**：Decimal128 类型需要使用特定的构造函数
   ```javascript
   // 正确方式
   db.finance.insertOne({ amount: NumberDecimal("123.45") })
   
   // 错误方式 (会存储为Double)
   db.finance.insertOne({ amount: 123.45 })
   ```

3. **索引效率**：不同类型的数字字段使用不同的索引效率，整数通常比浮点数索引效率更高