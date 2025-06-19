# MongoDB 中的 `new Date()` 用法详解

在 MongoDB 中，`new Date()` 用于创建日期对象，支持多种参数传递方式。下面我将详细介绍各种传参方式的细节和差异。

## 1. 不传参 - 当前日期时间

```javascript
new Date()
```
返回当前的日期和时间。

示例：
```
ISODate("2023-10-25T08:30:15.123Z")
```

## 2. 传递 ISO 日期字符串

```javascript
new Date("YYYY-MM-DDTHH:mm:ss.sssZ")
```

示例：
```javascript
new Date("2023-10-25T08:30:15.123Z")
```
返回：
```
ISODate("2023-10-25T08:30:15.123Z")
```

## 3. 传递日期时间组件

```javascript
new Date(year, monthIndex [, day [, hours [, minutes [, seconds [, milliseconds]]]]])
```

- `year`: 四位数的年份
- `monthIndex`: 0(一月)到11(十二月)
- 其他参数可选，默认为0

示例：
```javascript
new Date(2023, 9, 25, 8, 30, 15, 123)
```
返回：
```
ISODate("2023-10-25T08:30:15.123Z")
```

注意：monthIndex 是 9 表示十月(0-based)

## 4. 传递时间戳

```javascript
new Date(timestamp)
```

示例：
```javascript
new Date(1698222615123)
```
返回：
```
ISODate("2023-10-25T08:30:15.123Z")
```

## 5. 传递日期字符串(非ISO格式)

```javascript
new Date(dateString)
```

示例：
```javascript
new Date("October 25, 2023 08:30:15")
```
返回：
```
ISODate("2023-10-25T08:30:15Z")
```

## 各种传参方式对比

```markdown
| 传参方式               | 示例                                | 返回结果                              | 备注                          |
|------------------------|-----------------------------------|-------------------------------------|-----------------------------|
| 无参数                | `new Date()`                     | `ISODate("2023-10-25T08:30:15Z")`   | 返回当前日期时间               |
| ISO字符串             | `new Date("2023-10-25T08:30:15Z")` | `ISODate("2023-10-25T08:30:15Z")`   | 推荐使用ISO格式               |
| 日期时间组件          | `new Date(2023,9,25,8,30,15)`    | `ISODate("2023-10-25T08:30:15Z")`   | 月份是0-based(0=一月)         |
| 时间戳(毫秒)          | `new Date(1698222615000)`         | `ISODate("2023-10-25T08:30:15Z")`   | 自1970-01-01 UTC以来的毫秒数  |
| 非ISO日期字符串       | `new Date("Oct 25 2023")`        | `ISODate("2023-10-25T00:00:00Z")`   | 解析取决于实现，可能不一致     |
```

## 注意事项

1. MongoDB 存储日期为 UTC 时间，但在查询时会根据客户端时区进行转换
2. 在聚合管道中，可以使用 `$dateFromString` 将字符串转换为日期
3. 日期比较时，确保比较的是相同类型的对象(日期对象或日期字符串)

示例查询：
```javascript
db.collection.find({
  createdAt: {
    $gte: new Date("2023-10-01"),
    $lt: new Date("2023-11-01")
  }
})
```

这将查找 2023年10月的所有文档。