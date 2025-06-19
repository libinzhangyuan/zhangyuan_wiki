[返回](/mongodb/technolog/index)

# MongoDB 中的日期处理 (BSON Date)

MongoDB 使用 BSON 格式存储数据，其中日期类型被存储为特殊的 BSON Date 类型。以下是关于 MongoDB 中日期的详细介绍：

## BSON Date 类型特点

1. BSON Date 是一个 64 位整数，表示自 Unix 纪元(1970-01-01)以来的毫秒数
2. 在 MongoDB shell 中显示为 ISODate 格式
3. 时区信息不会被存储，日期以 UTC 格式存储

## 创建日期对象

### 1. 使用 `new Date()`

```javascript
// 创建当前日期
db.collection.insertOne({createdAt: new Date()})
```

返回结果示例：
```
{
  acknowledged: true,
  insertedId: ObjectId("5f8d88a3e4b6e3a9c8f3b2a1")
}
```

### 2. 使用 ISODate 字符串

```javascript
// 使用特定日期
db.collection.insertOne({
  event: "会议",
  date: new Date("2023-10-15T09:00:00Z")
})
```

返回结果示例：
```
{
  acknowledged: true,
  insertedId: ObjectId("5f8d88a3e4b6e3a9c8f3b2a2")
}
```

## 日期查询示例

### 1. 查询特定日期的文档

```javascript
db.events.find({
  date: {
    $gte: new Date("2023-10-01"),
    $lt: new Date("2023-10-31")
  }
})
```

返回结果示例：
```
[
  {
    "_id": ObjectId("5f8d88a3e4b6e3a9c8f3b2a2"),
    "event": "会议",
    "date": ISODate("2023-10-15T09:00:00Z")
  }
]
```

### 2. 使用日期运算符

```javascript
db.logs.find({
  timestamp: {
    $gt: new Date(Date.now() - 24 * 60 * 60 * 1000) // 过去24小时内的记录
  }
})
```

## 日期聚合操作

### 1. 按日期分组统计

```javascript
db.sales.aggregate([
  {
    $group: {
      _id: {
        year: { $year: "$date" },
        month: { $month: "$date" },
        day: { $dayOfMonth: "$date" }
      },
      total: { $sum: "$amount" }
    }
  }
])
```

返回结果示例：
```
[
  {
    "_id": {
      "year": 2023,
      "month": 10,
      "day": 15
    },
    "total": 1250
  }
]
```

### 2. 日期表达式操作符

MongoDB 提供了一系列日期操作符：
```
| 操作符 | 描述 | 示例 |
|--------|------|-------|
| `$year` | 提取年份 | `{ $year: "$date" }` |
| `$month` | 提取月份(1-12) | `{ $month: "$date" }` |
| `$dayOfMonth` | 提取月中日期(1-31) | `{ $dayOfMonth: "$date" }` |
| `$hour` | 提取小时(0-23) | `{ $hour: "$date" }` |
| `$minute` | 提取分钟(0-59) | `{ $minute: "$date" }` |
| `$second` | 提取秒(0-59) | `{ $second: "$date" }` |
| `$millisecond` | 提取毫秒(0-999) | `{ $millisecond: "$date" }` |
| `$dayOfYear` | 提取年中日期(1-366) | `{ $dayOfYear: "$date" }` |
| `$dayOfWeek` | 提取星期几(1-7, 1=周日) | `{ $dayOfWeek: "$date" }` |
| `$week` | 提取年中周数(0-53) | `{ $week: "$date" }` |
```
## 日期索引

为日期字段创建索引可以显著提高查询性能：

```javascript
db.logs.createIndex({ timestamp: 1 })
```

返回结果示例：
```
{
  createdCollectionAutomatically: false,
  numIndexesBefore: 1,
  numIndexesAfter: 2,
  ok: 1
}
```

## 日期与字符串的转换

### 1. 日期转字符串

```javascript
db.collection.aggregate([
  {
    $project: {
      dateString: {
        $dateToString: {
          format: "%Y-%m-%d",
          date: "$date"
        }
      }
    }
  }
])
```

返回结果示例：
```
[
  {
    "_id": ObjectId("5f8d88a3e4b6e3a9c8f3b2a2"),
    "dateString": "2023-10-15"
  }
]
```

### 2. 字符串转日期

```javascript
db.collection.insertOne({
  event: "转换示例",
  date: new Date("2023-10-20")
})
```

## 注意事项

1. MongoDB 日期总是以 UTC 格式存储
2. 应用层需要处理时区转换
3. 日期比较时要注意时区问题
4. 使用 `new Date()` 创建的是客户端当前时区的日期，但存储时会转换为 UTC

希望这些信息对您理解 MongoDB 中的日期处理有所帮助！