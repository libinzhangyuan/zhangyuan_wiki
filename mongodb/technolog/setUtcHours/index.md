[返回](/mongodb/technolog/index)

# MongoDB 中 `setUTCHours` 处理负数小时的情况

在 JavaScript 的 Date 对象中，当使用 `setUTCHours()` 方法设置的小时数小于 0 时，会自动向前一天调整，并相应地减少小时数。

## 处理机制说明

当 `setUTCHours()` 接收到负数小时参数时：
1. 每 -24 小时相当于向前调整 1 天
2. 剩余的小时数会从 23:00 开始倒减

## 示例演示

假设我们有以下 MongoDB 文档：

```json
[
  {
    "_id": 1,
    "event": "晨会",
    "utcTime": ISODate("2023-05-15T02:00:00Z")
  },
  {
    "_id": 2,
    "event": "午间休息",
    "utcTime": ISODate("2023-05-15T08:00:00Z")
  }
]
```

### 示例1：小时数减8后不小于0

```javascript
// 原始时间: 2023-05-15T08:00:00Z
let date = new Date("2023-05-15T08:00:00Z");
date.setUTCHours(date.getUTCHours() - 8);
```

结果：
```
2023-05-15T00:00:00Z
```

### 示例2：小时数减8后小于0

```javascript
// 原始时间: 2023-05-15T02:00:00Z
let date = new Date("2023-05-15T02:00:00Z");
date.setUTCHours(date.getUTCHours() - 8);
```

结果：
```
2023-05-14T18:00:00Z
```
（日期自动向前调整了一天）

## MongoDB 聚合中的等效操作

在 MongoDB 聚合管道中，可以使用 `$dateSubtract` 来实现类似功能：

```javascript
db.events.aggregate([
  {
    $project: {
      event: 1,
      originalTime: "$utcTime",
      adjustedTime: {
        $dateSubtract: {
          startDate: "$utcTime",
          unit: "hour",
          amount: 8
        }
      }
    }
  }
])
```

返回结果：

```
[
  {
    "_id": 1,
    "event": "晨会",
    "originalTime": ISODate("2023-05-15T02:00:00Z"),
    "adjustedTime": ISODate("2023-05-14T18:00:00Z")
  },
  {
    "_id": 2,
    "event": "午间休息",
    "originalTime": ISODate("2023-05-15T08:00:00Z"),
    "adjustedTime": ISODate("2023-05-15T00:00:00Z")
  }
]
```

## 对比表：JavaScript 与 MongoDB 日期调整

```
| 操作类型       | JavaScript 方式                    | MongoDB 聚合方式                  | 结果一致性 |
|---------------|-----------------------------------|-----------------------------------|-----------|
| 减8小时(正数) | date.setUTCHours(hours - 8)       | $dateSubtract(unit: "hour", amount: 8) | 相同      |
| 减8小时(跨日) | 自动调整日期                       | 自动调整日期                      | 相同      |
| 加8小时(跨日) | date.setUTCHours(hours + 8)       | $dateAdd(unit: "hour", amount: 8) | 相同      |
| 获取调整后小时 | date.getUTCHours()                | $hour 操作符                      | 相同      |
```

## 关键点总结

1. 当减小时数导致结果为负数时，`setUTCHours()` 会自动向前调整日期
2. 每24小时的负数调整相当于1天的日期变更
3. 在MongoDB聚合中，推荐使用 `$dateSubtract` 和 `$dateAdd` 而不是JavaScript方法
4. 两种方式处理跨日期的逻辑是一致的

如果您需要处理时区转换或其他复杂的日期操作，MongoDB 还提供了 `$timezone` 参数等更多功能。