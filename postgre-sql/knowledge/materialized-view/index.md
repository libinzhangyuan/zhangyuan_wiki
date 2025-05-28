[返回](/postgre-sql/knowledge/index)

# PostgreSQL 物化视图介绍

物化视图(Materialized View)是PostgreSQL中的一种数据库对象，它将查询结果持久化存储，可以显著提高复杂查询的性能。

## 物化视图的特点

1. **预先计算并存储结果**：与普通视图不同，物化视图会实际执行查询并将结果存储在磁盘上
2. **需要手动刷新**：数据不会自动更新，需要手动或定时刷新
3. **提高查询性能**：特别适合复杂聚合查询或大数据量查询
4. **占用存储空间**：因为存储了实际数据，会占用额外的磁盘空间

## 创建物化视图

基本语法：
```sql
CREATE MATERIALIZED VIEW view_name AS
SELECT columns
FROM tables
[WHERE conditions]
[GROUP BY group_by_expression]
[HAVING having_expression];
```

### 示例1：创建简单的物化视图

```sql
-- 创建一个包含产品分类统计的物化视图
CREATE MATERIALIZED VIEW product_category_stats AS
SELECT 
    category_id,
    COUNT(*) AS product_count,
    AVG(price) AS avg_price,
    MIN(price) AS min_price,
    MAX(price) AS max_price
FROM products
GROUP BY category_id;
```

## 查询物化视图

物化视图可以像普通表一样查询：

```sql
SELECT * FROM product_category_stats;
```

可能的返回结果：
```
 category_id | product_count |     avg_price     | min_price | max_price 
-------------+---------------+-------------------+-----------+-----------
           1 |            15 | 24.98666666666667 |      9.99 |     49.99
           2 |             8 | 89.98750000000000 |     29.99 |    199.99
           3 |            12 | 45.82416666666667 |     14.99 |     99.99
```

## 刷新物化视图

当基础表数据变化时，需要刷新物化视图以获取最新数据：

```sql
-- 完全刷新（重新计算整个视图）
REFRESH MATERIALIZED VIEW product_category_stats;

-- 使用CONCURRENTLY选项可以在刷新时不阻塞查询（需要唯一索引）
REFRESH MATERIALIZED VIEW CONCURRENTLY product_category_stats;
```

## 带索引的物化视图

可以为物化视图创建索引以提高查询性能：

```sql
-- 为物化视图创建唯一索引（CONCURRENT刷新需要）
CREATE UNIQUE INDEX idx_product_category_stats ON product_category_stats (category_id);

-- 创建其他索引
CREATE INDEX idx_avg_price ON product_category_stats (avg_price);
```

## 删除物化视图

```sql
DROP MATERIALIZED VIEW IF EXISTS product_category_stats;
```

## 实际应用示例

### 示例2：销售数据汇总

```sql
-- 创建销售汇总物化视图
CREATE MATERIALIZED VIEW sales_summary AS
SELECT 
    date_trunc('month', order_date) AS month,
    product_id,
    SUM(quantity) AS total_quantity,
    SUM(quantity * unit_price) AS total_revenue
FROM orders
JOIN order_items USING (order_id)
GROUP BY date_trunc('month', order_date), product_id;

-- 创建索引
CREATE UNIQUE INDEX idx_sales_summary ON sales_summary (month, product_id);
```

查询示例：
```sql
SELECT 
    month, 
    SUM(total_quantity) AS monthly_quantity,
    SUM(total_revenue) AS monthly_revenue
FROM sales_summary
GROUP BY month
ORDER BY month;
```

可能的返回结果：
```
         month          | monthly_quantity | monthly_revenue 
------------------------+------------------+-----------------
 2023-01-01 00:00:00+00 |             1250 |        24567.89
 2023-02-01 00:00:00+00 |             1420 |        28765.43
 2023-03-01 00:00:00+00 |             1680 |        32456.78
```

## 物化视图的优缺点

**优点**：
- 显著提高复杂查询的性能
- 减少重复计算的开销
- 可以像表一样建立索引

**缺点**：
- 数据不是实时的
- 需要手动刷新
- 占用额外存储空间
- 刷新可能消耗大量资源

## 适用场景

1. 数据仓库和报表系统
2. 聚合查询结果需要频繁访问
3. 基础数据不经常变化但查询频繁
4. 计算密集型查询

物化视图是PostgreSQL中优化查询性能的强大工具，特别适合读多写少、对实时性要求不高的场景。