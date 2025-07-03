[返回](/doris/knowledge/index)

# Doris 物化视图介绍

Doris 支持三种物化视图类型：ASYNC(异步)、MANUAL(手动)和INCREMENTAL(增量)，下面我将详细介绍这三种类型及其使用示例。

## 1. ASYNC 异步物化视图

异步物化视图是Doris中最常用的类型，系统会自动在后台异步刷新物化视图数据。

**特点：**
- 自动刷新(可配置刷新策略)
- 支持定时刷新和分区级刷新
- 适合大多数OLAP场景

**创建示例：**

```sql
CREATE MATERIALIZED VIEW store_sales_mv
DISTRIBUTED BY HASH(sale_date)
REFRESH ASYNC
AS 
SELECT 
    sale_date,
    store_id,
    SUM(sales_amount) AS total_sales,
    COUNT(*) AS sales_count
FROM store_sales
GROUP BY sale_date, store_id;
```

**查询物化视图：**

```sql
SELECT * FROM store_sales_mv ORDER BY sale_date LIMIT 5;
```

**返回结果示例：**

```
+------------+----------+-------------+-------------+
| sale_date  | store_id | total_sales | sales_count |
+------------+----------+-------------+-------------+
| 2023-01-01 |      101 |     4500.00 |          12 |
| 2023-01-01 |      102 |     3800.00 |           9 |
| 2023-01-02 |      101 |     5200.00 |          15 |
| 2023-01-02 |      102 |     4100.00 |          10 |
| 2023-01-03 |      101 |     4900.00 |          13 |
+------------+----------+-------------+-------------+
```

## 2. MANUAL 手动物化视图

手动物化视图需要用户显式地执行刷新命令来更新数据。

**特点：**
- 完全由用户控制刷新时机
- 适合数据更新不频繁的场景
- 可以避免不必要的刷新开销

**创建示例：**

```sql
CREATE MATERIALIZED VIEW user_behavior_mv
DISTRIBUTED BY HASH(user_id)
REFRESH MANUAL
AS
SELECT 
    user_id,
    COUNT(DISTINCT item_id) AS viewed_items,
    MAX(event_time) AS last_view_time
FROM user_behavior
WHERE behavior_type = 'view'
GROUP BY user_id;
```

**手动刷新命令：**

```sql
REFRESH MATERIALIZED VIEW user_behavior_mv;
```

**查询对比原始表和物化视图：**

原始表查询：
```sql
SELECT user_id, COUNT(DISTINCT item_id) 
FROM user_behavior 
WHERE behavior_type = 'view' 
GROUP BY user_id 
ORDER BY COUNT(DISTINCT item_id) DESC 
LIMIT 3;
```

```
+---------+----------------------+
| user_id | count(DISTINCT item_id) |
+---------+----------------------+
|  100231 |                  125 |
|  100145 |                  118 |
|  100378 |                  112 |
+---------+----------------------+
```

物化视图查询：
```sql
SELECT user_id, viewed_items 
FROM user_behavior_mv 
ORDER BY viewed_items DESC 
LIMIT 3;
```

```
+---------+-------------+
| user_id | viewed_items |
+---------+-------------+
|  100231 |         125 |
|  100145 |         118 |
|  100378 |         112 |
+---------+-------------+
```

## 3. INCREMENTAL 增量物化视图

增量物化视图只刷新新增或修改的数据，效率更高。

**特点：**
- 只刷新变更的分区数据
- 刷新速度快，资源消耗低
- 适合大规模数据且频繁更新的场景

**创建示例：**

```sql
CREATE MATERIALIZED VIEW sales_trend_mv
DISTRIBUTED BY HASH(product_id)
REFRESH INCREMENTAL
AS
SELECT 
    product_id,
    sale_date,
    SUM(quantity) AS total_quantity,
    SUM(amount) AS total_amount
FROM sales_detail
GROUP BY product_id, sale_date;
```

**查看物化视图刷新状态：**

```sql
SHOW MATERIALIZED VIEWS LIKE 'sales_trend_mv';
```

**返回结果示例：**

```
+----------------+-------------+----------------+----------------+---------+
| Id             | Name        | RefreshType    | LastRefreshTime| Status  |
+----------------+-------------+----------------+----------------+---------+
| 10045          | sales_trend_mv | INCREMENTAL   | 2023-06-15 14:30:00 | ACTIVE |
+----------------+-------------+----------------+----------------+---------+
```

## 三种物化视图类型对比

```
+------------------+------------------+------------------+---------------------+
| 特性             | ASYNC            | MANUAL           | INCREMENTAL         |
+------------------+------------------+------------------+---------------------+
| 刷新方式         | 自动             | 手动触发         | 自动增量            |
| 刷新粒度         | 全量或分区       | 全量             | 分区级增量          |
| 适用场景         | 常规OLAP         | 不频繁更新数据   | 频繁更新的大数据量  |
| 资源消耗         | 中等             | 低(可控)         | 低                  |
| 数据实时性       | 依赖刷新策略     | 完全可控         | 较高                |
| 创建语法复杂度   | 简单             | 简单             | 中等                |
+------------------+------------------+------------------+---------------------+
```

在实际使用中，可以根据业务需求选择合适的物化视图类型。ASYNC适合大多数分析场景，MANUAL适合不经常变化的数据，而INCREMENTAL则适合数据量大且更新频繁的场景。