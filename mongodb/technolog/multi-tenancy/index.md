[返回](/mongodb/technolog/index)

# MongoDB 多租户设计方案

多租户(Multi-tenancy)是指一个软件系统能够同时服务于多个客户(租户)，每个租户的数据和配置相互隔离。在MongoDB中实现多租户主要有以下几种方案：

## 1. 独立数据库模式

每个租户使用完全独立的数据库实例或独立的数据库。

**优点**：
- 完全隔离，安全性最高
- 性能影响最小
- 备份恢复简单

**缺点**：
- 资源消耗大
- 维护成本高

**示例代码**：
```javascript
// 连接租户A的数据库
const connA = MongoClient.connect("mongodb://localhost/tenant_a");
// 连接租户B的数据库
const connB = MongoClient.connect("mongodb://localhost/tenant_b");
```

## 2. 共享数据库，独立集合模式

所有租户共享同一个数据库，但每个租户有自己独立的集合，通常通过集合名前缀或后缀区分租户。

**示例**：
```javascript
// 插入租户A的数据
db.tenantA_users.insert({name: "张三", role: "admin"});
// 插入租户B的数据
db.tenantB_users.insert({name: "李四", role: "user"});
```

**查询结果示例**：
```
// 查询租户A的用户
db.tenantA_users.find()
```
```json
[
  {
    "_id": ObjectId("5f8d8a7b2f4d4e1d2c3b4a5c"),
    "name": "张三",
    "role": "admin"
  }
]
```

## 3. 共享集合，租户ID字段模式

所有租户共享相同的集合，通过一个租户ID字段区分数据。

**示例**：
```javascript
// 创建带tenant_id的文档
db.users.insert([
  {tenant_id: "A", name: "王五", email: "wangwu@a.com"},
  {tenant_id: "B", name: "赵六", email: "zhaoliu@b.com"}
]);
```

**查询特定租户数据**：
```javascript
db.users.find({tenant_id: "A"})
```
```json
[
  {
    "_id": ObjectId("5f8d8a7b2f4d4e1d2c3b4a5d"),
    "tenant_id": "A",
    "name": "王五",
    "email": "wangwu@a.com"
  }
]
```

## 4. 混合模式

结合独立集合和租户ID字段的模式，适用于大型系统。

## 方案对比

```markdown
| 方案                | 隔离级别 | 性能   | 扩展性 | 维护成本 | 适用场景               |
|---------------------|----------|--------|--------|----------|------------------------|
| 独立数据库          | 最高     | 最好   | 差     | 高       | 高安全性要求的金融系统 |
| 共享DB独立集合      | 高       | 好     | 中     | 中       | 中型SaaS应用           |
| 共享集合+租户ID字段 | 中       | 中     | 好     | 低       | 小型SaaS应用           |
| 混合模式            | 可调节   | 可调节 | 最好   | 中高     | 大型复杂系统           |
```

## 最佳实践建议

1. **索引优化**：确保tenant_id字段有索引
   ```javascript
   db.users.createIndex({tenant_id: 1});
   ```

2. **访问控制**：使用MongoDB角色控制租户数据访问
   ```javascript
   db.createRole({
     role: "tenantA_role",
     privileges: [{
       resource: {db: "appdb", collection: "tenantA_data"},
       actions: ["find", "update", "insert"]
     }],
     roles: []
   });
   ```

3. **数据归档**：定期归档不活跃租户数据
   ```javascript
   // 将6个月不活跃的租户数据移到归档集合
   db.users.aggregate([
     {$match: {last_active: {$lt: new Date(ISODate().getTime() - 180*24*60*60*1000)}}},
     {$out: "archived_users"}
   ]);
   ```

4. **分片策略**：大型系统可考虑按租户分片
   ```javascript
   sh.shardCollection("appdb.users", {tenant_id: 1});
   ```

选择哪种多租户方案应根据您的具体需求、租户数量、数据量和性能要求来决定。