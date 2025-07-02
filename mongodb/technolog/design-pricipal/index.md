[返回](/mongodb/technolog/index)

# MongoDB数据库设计经验法则

MongoDB作为一款流行的NoSQL数据库，其设计原则与传统关系型数据库有很大不同。以下是MongoDB数据库设计的一些重要经验法则：

## 1. 数据建模基本原则

### 1.1 根据查询模式设计集合结构

MongoDB设计应围绕应用程序的查询模式进行，而不是数据关系。

```json
// 不好的设计 - 过度规范化
// users集合
{
  "_id": 1,
  "name": "张三",
  "email": "zhang@example.com"
}

// orders集合
{
  "_id": 101,
  "user_id": 1,
  "items": ["手机", "耳机"],
  "total": 2999
}

// 好的设计 - 嵌入常用查询数据
{
  "_id": 1,
  "name": "张三",
  "email": "zhang@example.com",
  "recent_orders": [
    {
      "order_id": 101,
      "items": ["手机", "耳机"],
      "total": 2999,
      "date": ISODate("2023-05-01")
    }
  ]
}
```

### 1.2 预聚合数据

对于频繁计算的指标，应该在写入时预先计算好。

```json
// 不好的设计 - 需要实时计算
{
  "_id": 101,
  "user_id": 1,
  "items": ["手机", "耳机"],
  "total": 2999
}

// 好的设计 - 预聚合
{
  "_id": 1,
  "name": "张三",
  "order_count": 5,
  "total_spent": 12500,
  "last_order": ISODate("2023-05-01")
}
```

## 2. 嵌入 vs 引用

### 2.1 适合嵌入的情况
```
| 情况 | 示例 |
|------|------|
| 一对一关系 | 用户和用户档案 |
| 一对少关系 | 博客文章和评论 |
| 不频繁变化的子文档 | 产品规格 |
| 需要原子性更新的数据 | 购物车和商品 |
```
```json
// 嵌入示例 - 博客文章和评论
{
  "_id": 1001,
  "title": "MongoDB设计指南",
  "author": "李四",
  "comments": [
    {
      "user": "王五",
      "text": "很有帮助!",
      "date": ISODate("2023-05-10")
    },
    {
      "user": "赵六",
      "text": "期待更多示例",
      "date": ISODate("2023-05-12")
    }
  ]
}
```

### 2.2 适合引用的情况
```
| 情况 | 示例 |
|------|------|
| 一对多关系 | 用户和订单 |
| 多对多关系 | 学生和课程 |
| 大型子文档 | 产品评论 |
| 频繁更新的子文档 | 股票价格 |
```
```json
// 引用示例 - 用户和订单
// users集合
{
  "_id": 1,
  "name": "张三",
  "order_ids": [101, 102, 105]
}

// orders集合
{
  "_id": 101,
  "user_id": 1,
  "items": ["手机", "耳机"],
  "total": 2999
}
```

## 3. 模式设计策略

### 3.1 分桶模式

适用于时间序列数据，将数据按时间分桶存储。

```json
// 传感器数据分桶示例
{
  "_id": "sensor1_202305",
  "sensor_id": "sensor1",
  "month": "202305",
  "readings": [
    {
      "timestamp": ISODate("2023-05-01T00:00:00Z"),
      "value": 23.5
    },
    {
      "timestamp": ISODate("2023-05-01T00:01:00Z"),
      "value": 23.7
    }
    // 更多读数...
  ],
  "count": 1440,
  "avg_value": 24.2
}
```

### 3.2 属性模式

适用于具有大量可选字段的文档。

```json
// 产品属性示例
{
  "_id": "p100",
  "name": "智能手机",
  "attributes": [
    { "name": "颜色", "value": "黑色" },
    { "name": "内存", "value": "128GB" },
    { "name": "重量", "value": "189g" }
  ]
}
```

## 4. 性能优化法则

### 4.1 索引设计

- 为所有查询创建适当的索引
- 使用复合索引优化多字段查询
- 避免过度索引，影响写入性能

```json
// 创建复合索引示例
db.orders.createIndex({ user_id: 1, order_date: -1 })

// 索引覆盖查询示例
db.orders.find(
  { user_id: 1, order_date: { $gt: ISODate("2023-01-01") } },
  { _id: 0, order_id: 1, total: 1 }
)
```

### 4.2 分片策略
```
| 分片策略 | 适用场景 | 示例 |
|---------|---------|------|
| 范围分片 | 范围查询多 | 按日期分片 |
| 哈希分片 | 写入均匀分布 | 用户ID哈希分片 |
| 区域分片 | 数据局部性 | 按地理位置分片 |
```
## 5. 文档大小限制

- 单个文档最大16MB
- 考虑大文档的存储策略

```json
// 大文档处理示例 - 使用GridFS
// 存储文件元数据
{
  "_id": ObjectId("5f8d8a7d6f8a6d7a6d8a7d6"),
  "filename": "large_video.mp4",
  "length": 1024000000,
  "chunkSize": 261120,
  "uploadDate": ISODate("2023-05-15T10:12:35Z")
}

// 实际数据存储在fs.chunks集合中
```

## 6. 数据一致性考虑
```
- 写关注(Write Concern)级别设置
- 读偏好(Read Preference)设置
- 事务使用场景
```
```json
// 事务使用示例
session.startTransaction({
  readConcern: { level: "snapshot" },
  writeConcern: { w: "majority" }
});

try {
  db.accounts.updateOne(
    { _id: "A", balance: { $gte: 100 } },
    { $inc: { balance: -100 } },
    { session }
  );
  
  db.accounts.updateOne(
    { _id: "B" },
    { $inc: { balance: 100 } },
    { session }
  );
  
  session.commitTransaction();
} catch (error) {
  session.abortTransaction();
  throw error;
}
```

通过遵循这些经验法则，可以设计出高性能、可扩展的MongoDB数据库结构，满足应用程序的需求。