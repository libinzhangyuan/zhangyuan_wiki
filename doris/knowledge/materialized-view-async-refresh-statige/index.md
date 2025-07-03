# Doris 数据库 ASYNC 异步物化视图刷新策略介绍

ASYNC(异步)物化视图是 Apache Doris 中的一种物化视图类型，它通过后台任务定期刷新数据，与基表数据保持异步一致。

## ASYNC 物化视图特点

1. **异步刷新**：数据不会实时更新，而是在设定的时间间隔或满足条件时刷新
2. **资源友好**：刷新操作在后台进行，不影响前端查询性能
3. **支持多种刷新策略**：可以按时间间隔、定时或手动触发刷新

## 刷新策略类型

### 1. 定时刷新(REFRESH SCHEDULE)

```sql
CREATE MATERIALIZED VIEW mv_async_schedule
REFRESH ASYNC SCHEDULE START '2023-01-01 00:00:00' EVERY (INTERVAL 1 HOUR)
AS SELECT user_id, COUNT(*) as order_count FROM orders GROUP BY user_id;
```

### 2. 手动刷新(REFRESH MANUAL)

```sql
CREATE MATERIALIZED VIEW mv_async_manual
REFRESH ASYNC MANUAL
AS SELECT product_id, AVG(price) as avg_price FROM sales GROUP BY product_id;
```

## 刷新策略对比

```
| 刷新策略       | 语法示例                                | 刷新时机                 | 适用场景                     |
|----------------|----------------------------------------|--------------------------|----------------------------|
| SCHEDULE       | EVERY (INTERVAL 1 HOUR)                | 按固定时间间隔自动刷新   | 需要定期更新的汇总数据       |
| MANUAL         | MANUAL                                 | 用户显式执行刷新命令     | 不频繁更新的大型历史数据报表 |
```

## 刷新操作示例

### 查看物化视图刷新状态

```sql
SHOW ALTER MATERIALIZED VIEW\G;
```

示例返回结果：
```
*************************** 1. row ***************************
          JobId: 10086
      TableName: mv_async_schedule
     CreateTime: 2023-01-01 00:00:00
   FinishedTime: 2023-01-01 01:00:00
  BaseIndexName: orders
RollupIndexName: mv_async_schedule
       RollupId: 10010
  TransactionId: 5001
          State: FINISHED
      Msg: success
        Progress: 100%
```

### 手动触发刷新

```sql
REFRESH MATERIALIZED VIEW mv_async_manual;
```

示例返回结果：
```
Query OK, 0 rows affected (0.05 sec)
```

## 注意事项

1. 异步物化视图的数据与基表有一定延迟
2. 刷新过程会占用系统资源，建议在低峰期执行
3. 大型物化视图刷新可能需要较长时间
4. 可以通过 `SHOW MATERIALIZED VIEWS` 查看物化视图信息

```sql
SHOW MATERIALIZED VIEWS LIKE 'mv_async%';
```

示例返回结果：
```
+-------------------+-------------------+-------------------+------------+---------+
| Name              | Database          | Refresh Type      | Is Active  | Remarks |
+-------------------+-------------------+-------------------+------------+---------+
| mv_async_schedule | default           | ASYNC SCHEDULE    | true       |         |
| mv_async_manual   | default           | ASYNC MANUAL      | true       |         |
+-------------------+-------------------+-------------------+------------+---------+
```


