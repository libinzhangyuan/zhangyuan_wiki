# MongoDB findAndModify 原子性操作案例

`findAndModify` 的一个重要特性是它的原子性，这意味着查找和修改操作在单个原子步骤中完成，不会被其他操作中断。下面我将介绍几个典型的原子性操作案例。

## 1. 计数器/票务系统 - 安全递减

假设我们有一个票务系统，需要确保不会超卖票：

```javascript
// 票务集合初始数据
db.tickets.insert([
  { _id: "concert", total: 100, sold: 0, available: 100 }
])
```

原子性购票操作：

```javascript
db.tickets.findAndModify({
  query: { 
    _id: "concert",
    available: { $gt: 0 }  // 只有有余票时才操作
  },
  update: {
    $inc: {
      sold: 1,
      available: -1
    }
  },
  new: true
})
```

成功返回结果：

```
{
  "_id": "concert",
  "total": 100,
  "sold": 1,
  "available": 99
}
```

如果票已售罄，则返回 `null`。

## 2. 任务队列处理 - 获取并锁定任务

任务队列系统中常见的"获取并标记为处理中"模式：

```javascript
// 任务集合初始数据
db.tasks.insert([
  { _id: 1, status: "pending", details: "task1" },
  { _id: 2, status: "pending", details: "task2" }
])
```

原子获取任务：

```javascript
db.tasks.findAndModify({
  query: { status: "pending" },
  sort: { _id: 1 },  // 按ID顺序获取
  update: { $set: { status: "processing", worker: "worker1" } },
  new: true
})
```

返回结果：

```
{
  "_id": 1,
  "status": "processing",
  "details": "task1",
  "worker": "worker1"
}
```

## 3. 用户账户余额 - 安全转账

确保账户余额更新是原子性的：

```javascript
// 账户集合初始数据
db.accounts.insert([
  { _id: "user1", balance: 1000 },
  { _id: "user2", balance: 500 }
])
```

转账操作（从user1转100到user2）：

```javascript
// 第一步：从user1扣除金额
var result = db.accounts.findAndModify({
  query: { 
    _id: "user1",
    balance: { $gte: 100 }  // 确保余额足够
  },
  update: { $inc: { balance: -100 } },
  new: true
})

if (result) {
  // 第二步：只有扣款成功才给user2增加金额
  db.accounts.findAndModify({
    query: { _id: "user2" },
    update: { $inc: { balance: 100 } },
    new: true
  })
}
```

第一次操作返回结果：

```
{
  "_id": "user1",
  "balance": 900
}
```

## 4. 版本控制 - 乐观并发控制

实现文档版本控制，防止并发修改冲突：

```javascript
// 文档集合初始数据
db.documents.insert({
  _id: "doc1",
  title: "原始标题",
  content: "原始内容",
  version: 1
})
```

原子更新操作：

```javascript
db.documents.findAndModify({
  query: { 
    _id: "doc1",
    version: 1  // 只有版本为1时才更新
  },
  update: {
    $set: {
      title: "新标题",
      content: "新内容"
    },
    $inc: { version: 1 }
  },
  new: true
})
```

成功返回结果：

```
{
  "_id": "doc1",
  "title": "新标题",
  "content": "新内容",
  "version": 2
}
```

如果其他人已经修改了文档（版本不再是1），则操作失败返回 `null`。

## 5. 唯一序列号生成

生成不会重复的序列号：

```javascript
// 序列号计数器集合
db.counters.insert({
  _id: "orderId",
  seq: 0
})
```

原子获取下一个序列号：

```javascript
db.counters.findAndModify({
  query: { _id: "orderId" },
  update: { $inc: { seq: 1 } },
  new: true
})
```

返回结果：

```
{
  "_id": "orderId",
  "seq": 1
}
```

每次调用都会安全地返回一个唯一的递增值。

## 原子性操作对比表

```
| 场景                | 非原子操作的风险               | findAndModify解决方案          |
|---------------------|--------------------------------|--------------------------------|
| 票务销售            | 可能超卖票                    | 查询和减库存在一个原子操作完成 |
| 任务队列            | 同一任务可能被多个worker获取   | 获取和标记为处理中原子完成    |
| 账户转账            | 可能出现余额不一致             | 扣款和存款分为两个原子操作    |
| 并发修改            | 可能丢失更新                   | 使用版本号检查实现乐观锁      |
| 序列号生成          | 可能生成重复序号               | 递增操作保证原子性            |
```

## 关键点总结

1. `findAndModify` 的原子性确保了查询和更新操作不会被其他操作中断
2. 适用于需要先检查条件再更新的场景
3. 通过 `query` 条件可以实现各种业务约束检查
4. 返回修改前后的文档使得可以确认操作结果
5. 比先 `find` 再 `update` 的两步操作更安全可靠

这些案例展示了 `findAndModify` 在各种需要原子性操作的业务场景中的实际应用，特别是在需要确保数据一致性和防止竞争条件的系统中非常有用。