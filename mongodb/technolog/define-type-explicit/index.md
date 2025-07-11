[返回](/mongodb/technolog/index)

# MongoDB 显式类型指定函数

在MongoDB中，为了明确控制数据类型，特别是在数值类型上，提供了一系列类型构造函数。以下是主要的显式类型指定函数：

## 数值类型构造函数

```
| 函数名               | 说明                          | 对应BSON类型   | 示例                     |
|---------------------|-----------------------------|--------------|-------------------------|
| NumberInt()         | 创建32位整数                  | int          | NumberInt("42")         |
| NumberLong()        | 创建64位整数                  | long         | NumberLong("123456789") |
| NumberDecimal()     | 创建高精度十进制数             | decimal      | NumberDecimal("3.14159")|
| Double()            | 创建双精度浮点数(已不推荐使用)  | double       | Double(3.14)            |
```

### 示例1：数值类型使用

```javascript
// 插入包含不同类型数值的文档
db.products.insertOne({
  name: "高端显卡",
  price: NumberDecimal("999.99"),  // 高精度十进制，适合金融计算
  stock: NumberInt(50),            // 32位整数
  serial: NumberLong("1234567890123456789"), // 64位长整数
  discount: 0.15                   // 默认为双精度浮点
})
```

```
// 插入结果
{
  "_id": ObjectId("5f8d88d3e4b7e3a9d4e5f6a7"),
  "name": "高端显卡",
  "price": NumberDecimal("999.99"),
  "stock": 50,
  "serial": NumberLong("1234567890123456789"),
  "discount": 0.15
}
```

## 其他重要类型构造函数

```
| 函数名               | 说明                          | 对应BSON类型   | 示例                     |
|---------------------|-----------------------------|--------------|-------------------------|
| ObjectId()          | 创建MongoDB文档ID            | objectid     | ObjectId("507f1f77bcf86cd799439011") |
| Date()              | 创建日期对象                  | date         | new Date("2023-05-01")  |
| Timestamp()         | 创建时间戳                    | timestamp    | Timestamp(1, 123456789) |
| BinData()           | 创建二进制数据                | binData      | BinData(0, "SGVsbG8=")  |
| DBRef()             | 创建数据库引用                | dbref        | DBRef("collection", ObjectId("...")) |
| Code()              | 创建JavaScript代码           | javascript   | Code("function() { return true; }") |
| MinKey()            | 表示最小键值                  | minKey       | MinKey()                |
| MaxKey()            | 表示最大键值                  | maxKey       | MaxKey()                |
| RegExp()            | 创建正则表达式                | regex        | RegExp("^abc", "i")     |
```

### 示例2：特殊类型使用

```javascript
// 插入包含多种特殊类型的文档
db.systemLog.insertOne({
  _id: ObjectId("5f8d88d3e4b7e3a9d4e5f6a8"),
  event: "user_login",
  timestamp: new Timestamp(1, Math.floor(Date.now()/1000)),
  metadata: {
    ip: BinData(0, "MTI3LjAuMC4x"),  // "127.0.0.1"的base64编码
    userRef: DBRef("users", ObjectId("507f1f77bcf86cd799439011")),
    validation: Code("function() { return validateIP(this.ip); }")
  },
  createdAt: new Date(),
  priority: "high"
})
```

```
// 插入结果
{
  "_id": ObjectId("5f8d88d3e4b7e3a9d4e5f6a8"),
  "event": "user_login",
  "timestamp": Timestamp(1, 1672531200),
  "metadata": {
    "ip": BinData(0, "MTI3LjAuMC4x"),
    "userRef": DBRef("users", ObjectId("507f1f77bcf86cd799439011")),
    "validation": Code("function() { return validateIP(this.ip); }")
  },
  "createdAt": ISODate("2023-05-01T10:00:00Z"),
  "priority": "high"
}
```

## 类型检查与转换

MongoDB还提供了一些类型相关的操作符和函数：

```
| 操作符/函数          | 说明                          | 示例                     |
|---------------------|-----------------------------|-------------------------|
| $type               | 按字段类型查询                | { field: { $type: "string" } } |
| $convert            | 转换字段类型                  | { $convert: { input: "$age", to: "int" } } |
| $toInt              | 转换为整数                    | { $toInt: "$price" }    |
| $toLong             | 转换为长整数                  | { $toLong: "$timestamp" } |
| $toDouble           | 转换为双精度浮点数             | { $toDouble: "$rating" } |
| $toDecimal          | 转换为高精度十进制数           | { $toDecimal: "$total" } |
| $toString           | 转换为字符串                  | { $toString: "$_id" }   |
| $toDate             | 转换为日期                    | { $toDate: "$dateStr" } |
| $toBool             | 转换为布尔值                  | { $toBool: "$isActive" } |
```

### 示例3：类型转换使用

```javascript
// 聚合管道中的类型转换
db.orders.aggregate([
  {
    $project: {
      orderId: { $toString: "$_id" },
      total: { $toDecimal: "$total" },
      year: { $year: { $toDate: "$orderDate" } }
    }
  }
])
```

```
// 转换结果示例
{
  "orderId": "5f8d88d3e4b7e3a9d4e5f6a9",
  "total": NumberDecimal("123.45"),
  "year": 2023
}
```

## 注意事项

1. **数值精度**：
   - `NumberInt`：32位有符号整数 (-2,147,483,648 到 2,147,483,647)
   - `NumberLong`：64位有符号整数
   - `NumberDecimal`：128位十进制浮点数，适合金融计算

2. **类型转换规则**：
   - 字符串转数字时会尝试解析，失败则报错
   - 浮点数转整数会截断小数部分
   - 大数值转小类型可能导致溢出

3. **性能考虑**：
   - `NumberDecimal`计算比浮点类型慢
   - 适当使用`NumberInt`可以节省存储空间

合理使用这些显式类型函数可以确保数据的一致性和精确性，特别是在处理金融数据、科学计算和大整数等场景时尤为重要。