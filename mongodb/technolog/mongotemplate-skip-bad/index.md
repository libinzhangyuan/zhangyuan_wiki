# MongoDB 查询性能优化：MongoTemplate 在大集合中使用 skip 的性能问题

## 问题描述

当在大型集合中使用 `skip()` 方法时，MongoDB 的查询性能会显著下降。这是因为 MongoDB 的 `skip()` 操作需要扫描并跳过指定数量的文档，而不是直接跳转到目标位置。

## 性能问题原因

1. **全集合扫描**：`skip()` 需要遍历所有跳过的文档
2. **内存消耗**：大偏移量会导致大量数据被加载到内存
3. **无法使用索引优化**：与 SQL 的 `LIMIT OFFSET` 类似，`skip()` 不能有效利用索引

## 示例对比

### 慢查询示例（使用 skip）

```java
// 使用 MongoTemplate 查询，跳过前 100000 条记录
Query query = new Query().skip(100000).limit(10);
List<User> users = mongoTemplate.find(query, User.class);
```

对应的 MongoDB 查询日志：
```
[conn1] command test.users command: find { find: "users", filter: {}, skip: 100000, limit: 10 }
```

### 性能更好的替代方案

#### 方案1：使用范围查询 + 索引

```java
// 假设我们有一个按创建时间排序的索引
Query query = new Query()
    .addCriteria(Criteria.where("createdAt").gt(lastSeenDate))
    .limit(10);
List<User> users = mongoTemplate.find(query, User.class);
```

#### 方案2：使用游标或分页标记

```java
// 使用最后一个文档的ID作为下一页的起点
Query query = new Query()
    .addCriteria(Criteria.where("_id").gt(lastId))
    .limit(10);
List<User> users = mongoTemplate.find(query, User.class);
```

## 性能对比

```
| 方法               | 10,000条数据耗时 | 100,000条数据耗时 | 1,000,000条数据耗时 |
|--------------------|------------------|-------------------|---------------------|
| 使用skip()         | 120ms            | 1.2s              | 12s                 |
| 使用范围查询+索引  | 5ms              | 5ms               | 6ms                 |
| 使用游标/分页标记  | 4ms              | 4ms               | 5ms                 |
```

## 最佳实践建议

1. **避免大偏移量**：尽量不要使用大的 skip 值
2. **使用索引字段过滤**：改用 `$gt`、`$lt` 等范围查询操作符
3. **记录最后文档位置**：保存上一页最后一个文档的ID或排序字段值
4. **考虑分片集群**：对于超大集合，考虑使用分片来分散查询压力

## 实际案例

假设我们有一个用户集合，结构如下：

```json
{
  "_id": ObjectId("5f8d8a7b8c3a1b2c3d4e5f6"),
  "username": "user1",
  "createdAt": ISODate("2023-01-01T00:00:00Z"),
  "lastLogin": ISODate("2023-06-01T12:00:00Z")
}
```

### 不推荐的做法

```java
// 分页查询第100页，每页10条
int page = 100;
int size = 10;
Query query = new Query().skip((page - 1) * size).limit(size);
List<User> users = mongoTemplate.find(query, User.class);
```

### 推荐的做法

```java
// 基于最后看到的ID进行查询
ObjectId lastSeenId = ...; // 从上一页获取最后一个文档的ID
Query query = new Query()
    .addCriteria(Criteria.where("_id").gt(lastSeenId))
    .limit(10);
List<User> users = mongoTemplate.find(query, User.class);
```

通过这种方式，无论数据量多大，查询性能都能保持稳定。