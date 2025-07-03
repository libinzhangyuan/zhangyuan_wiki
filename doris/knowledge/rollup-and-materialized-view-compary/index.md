# Doris 中 Rollup 与物化视图的选择策略


在大多数现代Doris版本(1.2+)中，除非有非常明确的性能优势需求，否则通常建议优先使用物化视图，因为它更灵活且维护成本更低。只有在简单聚合场景且对延迟极度敏感时，才考虑使用Rollup。

在 Apache Doris 中，Rollup 和物化视图(Materialized View)都是预聚合技术，用于加速查询，但两者有不同的适用场景和优先级。

## 核心对比

```
| 特性                | Rollup                          | 物化视图                          |
|---------------------|--------------------------------|----------------------------------|
| 数据更新方式         | 与基表同步更新(强一致)          | 与基表同步更新(强一致)            |
| 存储方式            | 独立物理存储                    | 独立物理存储                      |
| 构建代价            | 较高(全量构建)                  | 较低(可增量)                      |
| 使用场景            | 固定维度组合的预聚合            | 灵活维度的预聚合                  |
| 查询改写            | 自动                            | 自动                              |
| 多表关联            | 不支持                          | 支持                              |
| 自定义表达式        | 有限支持                        | 完全支持                          |
| 维护成本            | 较高                            | 较低                              |
```

## 优先使用场景

### 优先使用 Rollup 的情况

```sql
-- 典型场景：固定维度的上卷操作(如从天到月汇总)
-- 创建Rollup示例
ALTER TABLE sales_detail 
ADD ROLLUP rollup_month(
    dt_month, 
    region, 
    SUM(cost), 
    COUNT(user_id)
)
KEY(dt_month, region)
FROM (
    SELECT DATE_TRUNC('month', dt) as dt_month, 
           region, 
           cost, 
           user_id
    FROM sales_detail
);
```

**适用条件**：
1. 维度组合固定且有限
2. 需要极低延迟的精确聚合
3. 聚合方式简单(SUM/COUNT/MIN/MAX等)
4. 单表场景

### 优先使用物化视图的情况

```sql
-- 典型场景：复杂聚合或多表关联
-- 创建物化视图示例
CREATE MATERIALIZED VIEW mv_order_analysis
DISTRIBUTED BY HASH(order_date) BUCKETS 10
REFRESH ASYNC
AS
SELECT 
    o.order_date,
    c.region,
    p.category,
    COUNT(o.order_id) as order_count,
    SUM(o.amount) as total_amount,
    AVG(o.amount) as avg_amount
FROM 
    orders o
    JOIN customers c ON o.customer_id = c.customer_id
    JOIN products p ON o.product_id = p.product_id
GROUP BY 
    o.order_date,
    c.region,
    p.category;
```

**适用条件**：
1. 需要跨表关联
2. 聚合逻辑复杂(包含表达式、CASE WHEN等)
3. 维度组合灵活多变
4. 可以接受秒级延迟
5. 需要高级聚合函数

## 决策流程图

```
是否需要跨表关联?
├── 是 → 必须使用物化视图
└── 否 → 维度组合是否固定?
    ├── 是 → 聚合是否简单?
    │   ├── 是 → 优先Rollup
    │   └── 否 → 使用物化视图
    └── 否 → 使用物化视图
```

## 实际案例对比

### 案例1：单表简单聚合

```sql
-- 原始表
CREATE TABLE user_behavior (
    dt DATETIME,
    user_id BIGINT,
    page_id INT,
    duration INT
)
DUPLICATE KEY(dt, user_id);

-- 方案1：Rollup
ALTER TABLE user_behavior 
ADD ROLLUP r_daily_summary (
    dt_date, 
    page_id, 
    SUM(duration), 
    COUNT(user_id)
)
KEY(dt_date, page_id)
FROM (
    SELECT DATE_TRUNC('day', dt) as dt_date, 
           page_id, 
           duration, 
           user_id
    FROM user_behavior
);

-- 方案2：物化视图
CREATE MATERIALIZED VIEW mv_daily_summary
DISTRIBUTE BY HASH(dt_date) BUCKETS 10
AS
SELECT 
    DATE_TRUNC('day', dt) as dt_date,
    page_id,
    SUM(duration) as total_duration,
    COUNT(DISTINCT user_id) as uv
FROM user_behavior
GROUP BY DATE_TRUNC('day', dt), page_id;

-- 查询性能对比(假设数据量1亿行)
```
```
| 查询类型          | Rollup响应时间 | 物化视图响应时间 |
|------------------|---------------|-----------------|
| 单日页面汇总      | 23ms          | 28ms            |
| 周汇总(7天扫描)   | 45ms          | 52ms            |
| 页面TopN查询      | 67ms          | 72ms            |
```

### 案例2：多表复杂聚合

```sql
-- 原始表
CREATE TABLE orders (...);
CREATE TABLE customers (...);
CREATE TABLE products (...);

-- 只能使用物化视图的方案
CREATE MATERIALIZED VIEW mv_sales_analysis
AS
SELECT 
    c.region,
    p.category,
    DATE_TRUNC('month', o.order_time) as month,
    COUNT(DISTINCT o.customer_id) as customer_count,
    SUM(CASE WHEN p.price > 1000 THEN o.amount ELSE 0 END) as premium_sales
FROM 
    orders o
    JOIN customers c ON o.customer_id = c.customer_id
    JOIN products p ON o.product_id = p.product_id
GROUP BY 
    c.region,
    p.category,
    DATE_TRUNC('month', o.order_time);
```

## 最佳实践建议

1. **混合使用策略**：
   - 对核心固定维度用Rollup
   - 对灵活分析需求用物化视图

2. **资源分配**：
   ```sql
   -- 为物化视图设置资源组
   ALTER MATERIALIZED VIEW mv_sales_analysis 
   SET ("resource_group" = "mv_group");
   ```

3. **监控维护**：
   ```sql
   -- 查看物化视图刷新状态
   SHOW ALTER MATERIALIZED VIEW FROM db_name;
   
   -- 查看Rollup构建进度
   SHOW ALTER TABLE ROLLUP FROM db_name;
   ```

4. **生命周期管理**：
   ```sql
   -- 自动过期不常用的物化视图
   ALTER MATERIALIZED VIEW mv_old_data 
   SET ("auto_refresh" = "false", "grace_period" = "7d");
   ```

在大多数现代Doris版本(1.2+)中，除非有非常明确的性能优势需求，否则通常建议优先使用物化视图，因为它更灵活且维护成本更低。只有在简单聚合场景且对延迟极度敏感时，才考虑使用Rollup。