[返回](/mongodb/technolog/index)

# MongoDB 跨分片查询详解

跨分片查询是指当查询条件不包含分片键或无法使用分片键精确路由时，MongoDB需要在多个分片上执行查询并将结果合并的过程。

## 跨分片查询类型

### 1. 定向查询 (Targeted Query)
当查询包含分片键的精确匹配或前缀匹配时，mongos可以将查询路由到特定分片。

```javascript
// 假设orders集合按customer_id分片
// 精确匹配分片键 - 定向查询
db.orders.find({ "customer_id": "C1001" })
```

### 2. 广播查询 (Broadcast Query)
当查询不包含分片键时，查询会被发送到所有分片。

```javascript
// 不包含分片键 - 广播查询
db.orders.find({ "amount": { "$gt": 100 } })
```

### 3. 部分定向查询 (Partial Targeted Query)
当查询使用分片键的部分前缀时，可能只定向到部分分片。

```javascript
// 复合分片键 {region:1, customer_id:1}
// 只使用前缀region - 部分定向
db.orders.find({ "region": "north" })
```

## 跨分片查询执行过程

1. mongos解析查询并确定需要访问哪些分片
2. 向相关分片发送查询请求
3. 各分片执行查询并返回结果
4. mongos合并结果并返回给客户端
5. 如有排序/限制/跳过等操作，mongos会执行这些操作

## 性能考虑

```
| 查询类型        | 性能影响                          | 优化建议                          |
|----------------|----------------------------------|----------------------------------|
| 定向查询        | 最佳性能，只访问必要分片          | 尽量使用分片键查询                |
| 广播查询        | 性能最差，访问所有分片            | 避免不使用分片键的查询            |
| 部分定向查询    | 中等性能，访问部分分片            | 设计合理的复合分片键              |
```

## 跨分片查询示例

### 示例1：广播查询

```javascript
// 不包含分片键的查询
db.orders.find({ "status": "shipped" }).explain("executionStats")
```

返回结果片段：
```
```json
{
  "queryPlanner": {
    "winningPlan": {
      "stage": "SHARD_MERGE",
      "shards": [
        {
          "shardName": "shard1",
          "executionStats": {
            "nReturned": 1245,
            "executionTimeMillis": 23
          }
        },
        {
          "shardName": "shard2",
          "executionStats": {
            "nReturned": 876,
            "executionTimeMillis": 18
          }
        }
      ]
    }
  },
  "executionStats": {
    "nReturned": 2121,
    "executionTimeMillis": 45
  }
}
```
```

### 示例2：排序和限制的跨分片查询

```javascript
// 跨分片排序查询
db.orders.find({ "amount": { "$gt": 100 } })
         .sort({ "order_date": -1 })
         .limit(10)
```

执行过程：
1. 所有分片执行 `{ "amount": { "$gt": 100 } }` 查询
2. 每个分片对自己的结果按 `order_date` 排序
3. 各分片返回前10个结果给mongos
4. mongos对所有分片返回的结果再次排序
5. 返回最终的前10个结果

## 优化跨分片查询的建议

1. **合理设计分片键**：选择查询频繁使用的字段作为分片键
2. **使用覆盖查询**：创建合适的索引使查询可以只使用索引
3. **限制返回数据量**：使用投影只返回必要字段
4. **考虑读写分布**：热点数据可能导致某些分片负载过高
5. **监控慢查询**：使用`db.currentOp()`和`explain()`分析慢查询

## 分片集群查询限制

1. 某些操作如`$text`搜索在分片集群中有特殊限制
2. 跨分片事务在MongoDB 4.2+才完全支持
3. `distinct`命令在分片集群中效率较低
4. `group`命令在分片集群中不可用，应改用聚合管道