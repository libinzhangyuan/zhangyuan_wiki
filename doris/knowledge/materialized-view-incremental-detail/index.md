[返回](/doris/knowledge/index)

# Doris物化视图的INCREMENTAL刷新模式详解

在Doris中，`INCREMENTAL`刷新模式是物化视图的一种高效刷新方式，它只刷新自上次刷新以来发生变化的数据。以下是关于INCREMENTAL刷新模式的详细说明：

## INCREMENTAL刷新模式的基本参数

### 1. 基本语法

```sql
CREATE MATERIALIZED VIEW mv_incremental
REFRESH INCREMENTAL
AS
SELECT columns FROM table WHERE condition;
```

### 2. 可选参数

INCREMENTAL模式支持以下关键参数：

#### a) 自动刷新间隔

```sql
REFRESH INCREMENTAL [START 'start_time'] EVERY (interval)
```

示例：
```sql
REFRESH INCREMENTAL START '2023-01-01 00:00:00' EVERY (INTERVAL 1 HOUR)
```

#### b) 手动刷新模式

```sql
REFRESH INCREMENTAL MANUAL
```

#### c) 分区刷新控制

```sql
REFRESH INCREMENTAL [PARTITION START ('partition1') END ('partition2')]
```

## 不同参数组合示例

### 示例1：自动增量刷新

```sql
CREATE MATERIALIZED VIEW mv_sales_daily
REFRESH INCREMENTAL START CURRENT_TIMESTAMP EVERY (INTERVAL 1 DAY)
AS
SELECT 
    date, 
    product_id, 
    SUM(amount) as total_sales
FROM sales
GROUP BY date, product_id;
```

### 示例2：手动增量刷新

```sql
CREATE MATERIALIZED VIEW mv_user_behavior
REFRESH INCREMENTAL MANUAL
AS
SELECT 
    user_id,
    COUNT(DISTINCT item_id) as viewed_items,
    MAX(event_time) as last_active
FROM user_events
WHERE event_type = 'view'
GROUP BY user_id;
```

### 示例3：分区增量刷新

```sql
CREATE MATERIALIZED VIEW mv_log_analysis
REFRESH INCREMENTAL PARTITION START ('202301') END ('202312')
AS
SELECT 
    month,
    log_level,
    COUNT(*) as count
FROM server_logs
GROUP BY month, log_level;
```

## INCREMENTAL刷新模式的特点

```
| 特性                | 说明                                                                 |
|---------------------|----------------------------------------------------------------------|
| 刷新效率            | 只处理变更数据，性能高                                               |
| 资源消耗            | 相比全量刷新，CPU和I/O消耗低                                         |
| 数据延迟            | 根据刷新间隔决定，可实现近实时                                       |
| 适用场景            | 基表频繁小量更新的场景                                               |
| 限制条件            | 需要表有明确的变更追踪机制(如版本号、时间戳等)                       |
```

## 查看INCREMENTAL刷新状态

```sql
SHOW MATERIALIZED VIEWS LIKE 'mv_sales_daily'\G
```

典型返回结果：
```
*************************** 1. row ***************************
               id: 10010
           name: mv_sales_daily
     refresh_type: INCREMENTAL
     is_active: true
    last_refresh_start_time: 2023-06-01 00:00:00
    last_refresh_finished_time: 2023-06-01 00:02:15
    last_refresh_duration: 135
    last_refresh_state: SUCCESS
    rows_count: 12500
    refresh_mode: INCREMENTAL
    incremental_refresh_scope: PARTITION
    incremental_refresh_partitions: p202306
```

## 注意事项

1. INCREMENTAL模式要求基表必须有：
   - 版本号字段
   - 或时间戳字段
   - 或明确的分区变更追踪

2. 不是所有查询都支持INCREMENTAL刷新，复杂聚合可能要求全量刷新

3. 可以通过`ALTER MATERIALIZED VIEW`修改刷新参数

4. 系统会自动记录每次刷新的元数据，用于确定增量范围