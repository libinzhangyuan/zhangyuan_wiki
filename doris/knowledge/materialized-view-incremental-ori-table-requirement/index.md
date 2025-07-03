[返回](/doris/knowledge/index)


# Doris物化视图INCREMENTAL刷新模式对基表的要求详解

INCREMENTAL(增量)刷新模式是Doris物化视图的高效刷新机制，但它要求基表必须满足特定条件才能正常工作。以下是三种支持增量刷新的基表要求的详细说明：

## 1. 版本号字段（Version Column）

### 要求特征
- 基表必须包含一个单调递增的版本号字段
- 该字段通常由系统自动维护（如Doris的`__DORIS_VERSION_COL__`）
- 每次数据更新都会自动增加版本号

### 适用场景
```sql
-- 创建包含版本号字段的表
CREATE TABLE products (
    id BIGINT,
    name VARCHAR(50),
    price DECIMAL(10,2),
    -- 其他字段...
    __DORIS_VERSION_COL__ BIGINT REPLACE DEFAULT "0"
)
UNIQUE KEY(id)
DISTRIBUTED BY HASH(id) BUCKETS 10;

-- 创建基于版本号的增量物化视图
CREATE MATERIALIZED VIEW mv_product_stats
REFRESH INCREMENTAL
AS
SELECT 
    category,
    COUNT(*) as product_count,
    AVG(price) as avg_price
FROM products
GROUP BY category;
```

### 工作原理
```
基表更新 → 版本号递增 → 物化视图通过比较版本号识别变更 → 仅刷新变更数据
```

## 2. 时间戳字段（Timestamp Column）

### 要求特征
- 基表必须包含记录最后修改时间的时间戳字段
- 字段类型需为DATETIME或TIMESTAMP
- 每次数据更新必须同步更新时间戳

### 适用场景
```sql
-- 创建包含时间戳字段的表
CREATE TABLE user_activities (
    user_id BIGINT,
    activity_type VARCHAR(20),
    activity_time DATETIME,
    last_update_time DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
)
DUPLICATE KEY(user_id, activity_time)
DISTRIBUTED BY HASH(user_id) BUCKETS 10;

-- 创建基于时间戳的增量物化视图
CREATE MATERIALIZED VIEW mv_user_activity_daily
REFRESH INCREMENTAL
AS
SELECT 
    DATE(activity_time) as day,
    activity_type,
    COUNT(*) as activity_count
FROM user_activities
GROUP BY day, activity_type;
```

### 工作原理
```
基表更新 → 更新时间戳 → 物化视图通过时间范围识别变更 → 仅刷新变更时间段数据
```

## 3. 分区变更追踪（Partition Change Tracking）

### 要求特征
- 基表必须是分区表
- 物化视图定义必须包含分区列
- 系统通过分区级别的变更来触发刷新

### 适用场景
```sql
-- 创建分区表
CREATE TABLE sales_records (
    order_id BIGINT,
    order_date DATE,
    customer_id BIGINT,
    amount DECIMAL(12,2)
)
PARTITION BY RANGE(order_date) (
    PARTITION p202301 VALUES LESS THAN ('2023-02-01'),
    PARTITION p202302 VALUES LESS THAN ('2023-03-01'),
    -- 其他分区...
)
DISTRIBUTED BY HASH(order_id) BUCKETS 10;

-- 创建基于分区变更的增量物化视图
CREATE MATERIALIZED VIEW mv_sales_monthly
REFRESH INCREMENTAL
AS
SELECT 
    DATE_TRUNC('month', order_date) as month,
    customer_id,
    SUM(amount) as total_amount
FROM sales_records
GROUP BY month, customer_id;
```

### 工作原理
```
新数据写入分区 → 系统标记分区变更 → 物化视图仅刷新变更分区 → 未变更分区数据不处理
```

## 三种机制对比

```
| 机制类型       | 字段要求                | 刷新粒度      | 适用场景                      | 限制条件                     |
|---------------|-------------------------|--------------|-----------------------------|----------------------------|
| 版本号字段    | 单调递增的版本字段      | 行级         | 频繁单行更新的表             | 需要REPLACE聚合模型         |
| 时间戳字段    | 最后更新时间字段        | 时间范围级   | 按时间分布的时序数据         | 必须保证时间戳准确更新      |
| 分区变更追踪  | 分区表+分区列           | 分区级       | 按分区管理的大表             | 物化视图需包含分区列        |
```

## 检查表是否支持INCREMENTAL刷新

可以通过EXPLAIN命令验证：

```sql
EXPLAIN CREATE MATERIALIZED VIEW mv_test
REFRESH INCREMENTAL
AS SELECT * FROM your_table;
```

如果返回计划中包含"IncrementalRefresh"字样，则表示支持增量刷新。

## 注意事项

1. **混合使用**：可以同时使用多种机制（如版本号+分区追踪）
2. **DDL影响**：修改表结构可能导致增量刷新失效
3. **数据一致性**：确保时间戳/版本号准确更新，否则会导致数据不一致
4. **性能权衡**：行级增量刷新可能比分区级消耗更多资源

正确配置增量刷新机制可以显著提高物化视图刷新效率，降低资源消耗。