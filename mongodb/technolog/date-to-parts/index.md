在 MongoDB 中，`$dayOfMonth` 用于提取日期字段的“日”部分（1-31）。但默认情况下，它使用 **UTC 时间**，如果你的数据涉及不同时区，可能需要调整时区计算。  

---

## **1. `$dayOfMonth` 的默认行为（UTC）**
假设有一个 `events` 集合，存储不同时间的事件：
```javascript
db.events.insertMany([
  { _id: 1, eventTime: ISODate("2023-10-15T23:30:00Z") },  // UTC 时间
  { _id: 2, eventTime: ISODate("2023-11-03T01:30:00+08:00") }  // 东八区时间
]);
```
使用 `$dayOfMonth` 提取日部分：
```javascript
db.events.aggregate([
  {
    $project: {
      day: { $dayOfMonth: "$eventTime" }
    }
  }
]);
```
### **返回结果（UTC 计算）**
```
[
  { "_id": 1, "day": 15 },  // 2023-10-15T23:30:00Z → 15
  { "_id": 2, "day": 2 }    // 2023-11-03T01:30:00+08:00 → UTC 时间 2023-11-02T17:30:00Z → 2
]
```
**问题**：  
- 第 2 条记录的 `eventTime` 是 `2023-11-03 01:30 +08:00`（北京时间）
，但 MongoDB 默认按 **UTC** 计算，所以实际返回的是 `2`
（因为 UTC 时间是 `2023-11-02 17:30`）。

---

## **2. 调整时区：`$dateToString` + `timezone`**
如果想按 **特定时区** 计算 `$dayOfMonth`，可以先用 `$dateToString` 转换时区，再提取日部分：
```javascript
db.events.aggregate([
  {
    $project: {
      localDate: {
        $dateToString: {
          format: "%Y-%m-%d",
          date: "$eventTime",
          timezone: "+08:00"  // 东八区（北京时间）
        }
      }
    }
  },
  {
    $project: {
      day: { $dayOfMonth: { $toDate: "$localDate" } }
    }
  }
]);
```
### **返回结果（东八区计算）**
```
[
  { "_id": 1, "day": 16 },  // UTC 2023-10-15T23:30:00Z → 北京时间 2023-10-16T07:30:00 → 16
  { "_id": 2, "day": 3 }    // 2023-11-03T01:30:00+08:00 → 3
]
```

```
**说明**：
- 第 1 条记录的 UTC 时间是 `2023-10-15 23:30`，转换为 **北京时间（+08:00）** 是 `2023-10-16 07:30`，所以 `$dayOfMonth` 返回 `16`。
- 第 2 条记录已经是 **东八区时间**，所以 `$dayOfMonth` 正确返回 `3`。
```

---

## **3. 更简单的方法：`$dateToParts` + `timezone`（MongoDB 5.0+）**
在 MongoDB 5.0 及以上版本，可以直接使用 `$dateToParts` 并指定时区：
```javascript
db.events.aggregate([
  {
    $project: {
      dateParts: {
        $dateToParts: {
          date: "$eventTime",
          timezone: "+08:00"  // 东八区
        }
      }
    }
  },
  {
    $project: {
      day: "$dateParts.day"
    }
  }
]);
```
### **返回结果**
```
[
  { "_id": 1, "day": 16 },
  { "_id": 2, "day": 3 }
]
```

---
```
## **4. 对比表**
| 方法 | 适用版本 | 优点 | 缺点 |
|------|---------|------|------|
| **`$dayOfMonth`（默认 UTC）** | 所有版本 | 简单 | 无法处理时区 |
| **`$dateToString` + 时区** | 所有版本 | 兼容性好 | 需要额外转换 |
| **`$dateToParts` + 时区** | MongoDB 5.0+ | 最直接 | 需要较新版本 |
```
---

## **总结**
1. **默认 `$dayOfMonth` 使用 UTC**，可能导致时区问题。
2. **调整时区的方法**：
   - **`$dateToString` + `timezone`**（适用于所有版本）
   - **`$dateToParts` + `timezone`**（MongoDB 5.0+ 推荐）
3. **如果数据涉及多时区**，建议存储时统一使用 **UTC**，查询时再转换时区。