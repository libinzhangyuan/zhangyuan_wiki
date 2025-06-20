在 MongoDB 中，数值类型默认会以 `double`（双精度浮点数）的形式存储，除非显式指定为整数类型。因此，当你执行 `db.test.insertOne({ value: 5 })` 时，MongoDB 默认会存储为 `5.0`（即浮点数），而不是整数 `5`。

### 验证方法：
你可以通过插入数据后查询其类型来验证：

```javascript
// 插入数据
db.test.insertOne({ value: 5 });

// 查询并显示数据类型
var doc = db.test.findOne({ value: 5 });
typeof doc.value;
```

### 返回结果示例：
```
number
```

### 进一步验证：
如果你希望明确存储为整数，可以使用 `NumberInt()` 函数：

```javascript
// 显式存储为整数
db.test.insertOne({ value: NumberInt(5) });

// 查询并检查类型
var doc = db.test.findOne({ value: NumberInt(5) });
Object.prototype.toString.call(doc.value);
```

### 返回结果示例：
```
[object NumberInt]
```

### 对比表：
```
| 插入方式 | 存储类型 | 示例值 |
|----------|----------|--------|
| `{ value: 5 }` | double（浮点数） | `5.0` |
| `{ value: NumberInt(5) }` | 32位整数 | `5` |
```
### 总结：
- 直接写 `{ value: 5 }`，MongoDB 会存储为 `double` 类型的 `5.0`。
- 如果需要存储为整数，需显式使用 `NumberInt(5)`。