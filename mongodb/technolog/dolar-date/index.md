# MongoDB 中的 `$date` 操作符详解

`$date` 在 MongoDB 中有两种主要用法，取决于上下文环境：

## 1. 在聚合管道中的 `$date` 操作符

在聚合框架中，`$date` 是一个日期转换操作符，用于将输入值转换为日期对象。

### 语法
```javascript
{ $date: <expression> }
```

### 功能
- 将符合日期格式的字符串转换为日期对象
- 将表示毫秒数的数字转换为日期对象
- 如果输入已经是日期对象，则直接返回该对象

### 示例

```javascript
db.orders.aggregate([
  {
    $project: {
      orderId: 1,
      // 将字符串转换为日期
      orderDate: { $date: "$dateString" },
      // 将毫秒数转换为日期
      processedAt: { $date: "$timestampMillis" }
    }
  }
])
```

假设集合中有如下文档：
```json
{
  "_id": 1,
  "dateString": "2023-05-15T10:00:00Z",
  "timestampMillis": 1684144800000
}
```

返回结果：
```json
{
  "_id": 1,
  "orderId": "ORD123",
  "orderDate": ISODate("2023-05-15T10:00:00Z"),
  "processedAt": ISODate("2023-05-15T10:00:00Z")
}
```

## 2. 在查询表达式中的 `$date` 操作符

在查询表达式中，`$date` 通常用于日期字段的引用或日期字面量的创建。

### 示例1：引用日期字段
```javascript
db.events.find({
  $expr: { $eq: [{ $year: "$date" }, 2023] }
})
```
这里的 `$date` 引用了文档中的 `date` 字段

### 示例2：创建日期字面量
```javascript
db.logs.find({
  createdAt: { $gt: { $date: "2023-01-01T00:00:00Z" } }
})
```

## 3. 日期字符串格式

`$date` 操作符接受的字符串格式包括：
- ISO 8601 格式：`"YYYY-MM-DDTHH:MM:SSZ"`
- 简写格式：`"YYYY-MM-DD"`

## 4. 与其他日期操作符的配合使用

`$date` 常与其他日期操作符一起使用：

```javascript
db.analytics.aggregate([
  {
    $project: {
      eventDate: { $date: "$eventString" },
      year: { $year: { $date: "$eventString" } },
      month: { $month: { $date: "$eventString" } }
    }
  }
])
```

## 5. 错误处理

如果输入无法转换为有效日期，`$date` 会报错。可以使用 `$convert` 进行更安全的转换：

```javascript
db.collection.aggregate([
  {
    $project: {
      safeDate: {
        $convert: {
          input: "$dateString",
          to: "date",
          onError: null,
          onNull: null
        }
      }
    }
  }
])
```

## 实际应用场景

### 场景1：处理混合格式的日期数据

```javascript
db.records.aggregate([
  {
    $project: {
      recordId: 1,
      // 统一转换各种格式的日期
      effectiveDate: {
        $cond: {
          if: { $eq: [{ $type: "$dateField" }, "string"] },
          then: { $date: "$dateField" },
          else: "$dateField"
        }
      }
    }
  }
])
```

### 场景2：计算日期差值

```javascript
db.orders.aggregate([
  {
    $project: {
      orderId: 1,
      orderDate: { $date: "$orderDateStr" },
      deliveryDate: { $date: "$deliveryDateStr" },
      deliveryDays: {
        $divide: [
          { $subtract: [
            { $date: "$deliveryDateStr" },
            { $date: "$orderDateStr" }
          ]},
          1000 * 60 * 60 * 24 // 转换为天数
        ]
      }
    }
  }
])
```

## 注意事项

1. 时区问题：`$date` 转换时会使用 UTC 时区
2. 性能考虑：在大型集合上使用 `$date` 转换可能影响性能
3. 数据一致性：确保所有文档中的日期字段格式一致

`$date` 操作符是 MongoDB 中处理日期数据的重要工具，特别适合用于数据清洗和格式统一化处理。