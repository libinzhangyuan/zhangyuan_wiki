# MongoDB 游标关闭指南

在 MongoDB 中，游标(Cursor)是查询结果集的指针，正确管理游标对资源利用和性能很重要。下面详细介绍如何及时关闭 MongoDB 游标。

## 为什么要关闭游标

1. 释放服务器资源
2. 避免内存泄漏
3. 提高应用程序性能

## 自动关闭游标的情况

在以下情况下，MongoDB 会自动关闭游标：

1. 游标遍历完所有结果后
2. 游标超过 10 分钟不活动
3. 客户端与服务器连接断开

## 手动关闭游标的方法

### 1. 显式调用 close() 方法

```javascript
const cursor = db.collection('users').find({age: {$gt: 30}});

// 遍历部分结果
for (let i = 0; i < 5 && cursor.hasNext(); i++) {
    printjson(cursor.next());
}

// 显式关闭游标
cursor.close();
```

### 2. 使用 try-finally 确保关闭

```javascript
let cursor;
try {
    cursor = db.collection('orders').find({status: "completed"});
    // 处理结果
    cursor.forEach(printjson);
} finally {
    if (cursor) {
        cursor.close();
    }
}
```

### 3. 使用 JavaScript 的迭代语法

现代 MongoDB 驱动支持 async/await 语法，可以自动管理游标：

```javascript
async function processUsers() {
    const cursor = db.collection('users').find();
    for await (const doc of cursor) {
        // 处理文档
        console.log(doc);
    }
    // 游标会自动关闭
}
```

## 游标使用最佳实践

1. **尽早关闭**：一旦不再需要游标，立即关闭它
2. **批量处理**：使用 `batchSize()` 控制每次从服务器获取的文档数量
3. **限制结果**：结合 `limit()` 使用以减少返回的文档数

```javascript
// 最佳实践示例
const cursor = db.collection('products')
    .find({category: "electronics"})
    .limit(100)
    .batchSize(20);

try {
    cursor.forEach(product => {
        // 处理产品
    });
} finally {
    cursor.close();
}
```

## 游标状态检查

你可以检查游标是否已关闭：

```javascript
const cursor = db.collection('logs').find();
print(cursor.isClosed());  // 输出 false
cursor.close();
print(cursor.isClosed());  // 输出 true
```

返回结果示例：
```
false
true
```

## 不同语言驱动的关闭方法对比

```
| 语言       | 关闭游标方法                     | 自动关闭情况                     |
|------------|----------------------------------|----------------------------------|
| JavaScript | cursor.close()                  | for-await-of 循环结束后自动关闭  |
| Python     | cursor.close()                  | with 语句块结束后自动关闭        |
| Java       | cursor.close()                  | try-with-resources 自动关闭      |
| C#         | cursor.Dispose()                | using 语句块结束后自动关闭       |
| PHP        | $cursor->dead = true            | 无自动关闭机制                   |
```

通过遵循这些实践，你可以有效地管理 MongoDB 游标，避免资源泄漏和性能问题。