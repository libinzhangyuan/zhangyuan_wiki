[返回](/doris/knowledge/index)

# Doris 数据库中的 ROLLUP 和物化视图介绍

在 Apache Doris 中，ROLLUP 是一种重要的预聚合技术，用于提高查询性能。下面我将详细介绍 ROLLUP 以及与之类似的物化视图概念。

## 1. ROLLUP 介绍

ROLLUP 是 Doris 中的一种预聚合方式，它会在基表的基础上按照指定的维度列进行预聚合，生成预聚合表。当查询命中这些预聚合表时，可以直接使用预计算好的结果，大大提升查询效率。

### ROLLUP 示例

假设我们有一个销售数据表：

```sql
CREATE TABLE sales (
    dt DATE,
    region VARCHAR(50),
    product VARCHAR(50),
    amount DECIMAL(10,2)
)
DISTRIBUTED BY HASH(region) BUCKETS 10
PROPERTIES (
    "replication_num" = "3"
);
```

我们可以为这个表创建一个 ROLLUP：

```sql
ALTER TABLE sales ADD ROLLUP rollup_region_product(dt, region, product, amount);
```

查询原始表和 ROLLUP 的区别：

```sql
-- 查询原始表
SELECT region, product, SUM(amount) 
FROM sales 
WHERE dt BETWEEN '2023-01-01' AND '2023-01-31'
GROUP BY region, product;

-- 查询结果示例：
```
```
+--------+---------+-------------+
| region | product | sum(amount) |
+--------+---------+-------------+
| 华东   | 手机    | 125000.00   |
| 华北   | 电脑    | 98000.00    |
| 华南   | 平板    | 75600.00    |
+--------+---------+-------------+
```

```sql
-- 当查询命中 ROLLUP 时，Doris 会自动使用预聚合数据
EXPLAIN SELECT region, product, SUM(amount) 
FROM sales 
WHERE dt BETWEEN '2023-01-01' AND '2023-01-31'
GROUP BY region, product;

-- 执行计划会显示使用了 rollup_region_product
```

## 2. 物化视图 (Materialized View)

物化视图是 Doris 中另一种预聚合方式，比 ROLLUP 更灵活，可以支持更复杂的聚合函数和表达式。

### 物化视图示例

创建物化视图：

```sql
CREATE MATERIALIZED VIEW mv_region_monthly
DISTRIBUTED BY HASH(region)
REFRESH ASYNC
AS
SELECT 
    region,
    DATE_TRUNC('month', dt) AS month,
    SUM(amount) AS total_amount,
    COUNT(*) AS order_count
FROM sales
GROUP BY region, DATE_TRUNC('month', dt);
```

查询对比：

```sql
-- 原始查询
SELECT 
    region,
    DATE_TRUNC('month', dt) AS month,
    SUM(amount) AS total_amount,
    COUNT(*) AS order_count
FROM sales
WHERE dt BETWEEN '2023-01-01' AND '2023-03-31'
GROUP BY region, DATE_TRUNC('month', dt);

-- 结果示例：
```
```
+--------+------------+--------------+-------------+
| region | month      | total_amount | order_count |
+--------+------------+--------------+-------------+
| 华东   | 2023-01-01 | 356000.00    | 1200        |
| 华北   | 2023-02-01 | 289000.00    | 950         |
| 华南   | 2023-03-01 | 412000.00    | 1350        |
+--------+------------+--------------+-------------+
```

```sql
-- 当查询命中物化视图时
EXPLAIN SELECT 
    region,
    DATE_TRUNC('month', dt) AS month,
    SUM(amount) AS total_amount
FROM sales
WHERE dt BETWEEN '2023-01-01' AND '2023-03-31'
GROUP BY region, DATE_TRUNC('month', dt);

-- 执行计划会显示使用了 mv_region_monthly
```

## 3. ROLLUP 和物化视图对比

```
+----------------------+--------------------------+----------------------------------+
| 特性                | ROLLUP                   | 物化视图                        |
+----------------------+--------------------------+----------------------------------+
| 创建方式            | ALTER TABLE ADD ROLLUP   | CREATE MATERIALIZED VIEW        |
| 聚合灵活性          | 简单聚合                 | 支持复杂表达式和多种聚合函数     |
| 刷新机制            | 自动与基表同步           | 可配置ASYNC/MANUAL/INCREMENTAL  |
| 存储开销            | 较低                     | 较高                            |
| 查询改写            | 自动                     | 自动                            |
| 适用场景            | 简单预聚合场景           | 复杂预聚合和计算场景            |
+----------------------+--------------------------+----------------------------------+
```

## 4. 使用建议

1. 对于简单的预聚合需求，优先考虑 ROLLUP，因为它更轻量级
2. 当需要复杂计算或自定义表达式时，使用物化视图
3. 物化视图适合对实时性要求不高的场景，可以配置异步刷新
4. 监控预聚合对象的使用情况，定期清理不常用的预聚合

通过合理使用 ROLLUP 和物化视图，可以显著提升 Doris 的查询性能，特别是在处理大规模数据分析时。