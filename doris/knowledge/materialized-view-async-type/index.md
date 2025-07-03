[返回](/doris/knowledge/index)


# Doris物化视图创建时的REFRESH ASYNC模式区别

在Doris中创建物化视图时，可以通过`REFRESH ASYNC`指定不同的刷新策略。以下是三种创建方式的详细区别：

## 1. 基本异步刷新模式

```sql
CREATE MATERIALIZED VIEW mv_async_manual
REFRESH ASYNC
AS
SELECT user_id, COUNT(*) as order_count, SUM(amount) as total_amount
FROM orders
GROUP BY user_id;
```

**特点**：
- 创建时不自动刷新，需要手动触发第一次刷新
- 后续也需要手动执行`REFRESH ASYNC MATERIALIZED VIEW`来更新
- 不会自动调度刷新任务

## 2. 定时异步刷新模式

```sql
CREATE MATERIALIZED VIEW mv_async_schedule
REFRESH ASYNC SCHEDULE START '2023-01-01 00:00:00' EVERY (INTERVAL 1 HOUR)
AS
SELECT user_id, COUNT(*) as order_count, SUM(amount) as total_amount
FROM orders
GROUP BY user_id;
```

**特点**：
- 自动从指定时间开始按计划刷新(本例每小时一次)
- 系统会自动维护刷新任务
- 可以通过`ALTER MATERIALIZED VIEW`修改调度计划

## 3. 纯手动刷新模式

```sql
CREATE MATERIALIZED VIEW mv_async_manual
REFRESH ASYNC MANUAL
AS
SELECT user_id, COUNT(*) as order_count, SUM(amount) as total_amount
FROM orders
GROUP BY user_id;
```

**特点**：
- 完全依赖手动刷新
- 不会自动创建任何刷新任务
- 明确表示该物化视图不需要自动刷新

## 三种创建方式对比

```
| 创建方式                      | 首次刷新 | 自动刷新 | 适用场景                         |
|------------------------------|----------|----------|----------------------------------|
| REFRESH ASYNC                | 需手动   | 无       | 不确定刷新频率，需要灵活控制     |
| REFRESH ASYNC SCHEDULE       | 自动     | 有       | 需要定期更新数据的物化视图       |
| REFRESH ASYNC MANUAL         | 需手动   | 无       | 明确不需要自动刷新的物化视图     |

REFRESH ASYNC MANUAL 与 REFRESH ASYNC  效果是相同的
```

## 查看物化视图刷新信息

创建后可以查看物化视图的刷新设置：

```sql
SHOW CREATE MATERIALIZED VIEW mv_async_schedule;
```

示例返回：
```
*************************** 1. row ***************************
       Materialized View: mv_async_schedule
       Create Materialized View: CREATE MATERIALIZED VIEW `mv_async_schedule`
REFRESH ASYNC SCHEDULE START '2023-01-01 00:00:00' EVERY (INTERVAL 1 HOUR)
AS SELECT user_id, COUNT(*) as order_count, SUM(amount) as total_amount
FROM orders
GROUP BY user_id
```

## 修改刷新策略示例

可以将手动刷新的物化视图改为定时刷新：

```sql
ALTER MATERIALIZED VIEW mv_async_manual 
REFRESH ASYNC SCHEDULE START CURRENT_TIMESTAMP EVERY (INTERVAL 1 DAY);
```