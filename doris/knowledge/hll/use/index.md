[返回](/doris/knowledge/hll/index)

# Doris数据库中的HLL（HyperLogLog）使用指南

HLL（HyperLogLog）是Doris数据库中用于基数统计的一种高级功能，它可以高效地估算大规模数据集的基数（不重复元素的数量），同时占用极小的存储空间。

## 1. HLL简介

HLL是一种概率数据结构，用于估算大量数据的基数。它的特点是：
- 估算误差率约为1%~2%
- 占用空间极小（Doris中默认最大16KB）
- 支持合并操作，适合分布式计算

## 2. HLL基本操作

### 2.1 创建包含HLL列的表

```sql
CREATE TABLE user_visits (
    date DATE,
    user_id HLL HLL_UNION
) 
ENGINE=OLAP
DISTRIBUTED BY HASH(date) BUCKETS 10;
```

### 2.2 插入数据到HLL列

```sql
-- 使用hll_hash函数将原始数据转换为HLL
INSERT INTO user_visits VALUES ('2023-01-01', hll_hash('user1'));
INSERT INTO user_visits VALUES ('2023-01-01', hll_hash('user2'));

-- 或者批量插入
INSERT INTO user_visits 
VALUES ('2023-01-02', hll_hash('user1')),
       ('2023-01-02', hll_hash('user3'));
```

### 2.3 查询HLL数据

```sql
-- 使用hll_cardinality函数获取估算基数
SELECT date, hll_cardinality(user_id) AS uv 
FROM user_visits;
```

返回结果示例：
```
+------------+------+
| date       | uv   |
+------------+------+
| 2023-01-01 | 2    |
| 2023-01-02 | 2    |
+------------+------+
```

## 3. HLL聚合操作

### 3.1 合并HLL值

```sql
-- 使用hll_union_agg函数合并多行HLL值
SELECT hll_cardinality(hll_union_agg(user_id)) AS total_uv
FROM user_visits;
```

返回结果示例：
```
+-----------+
| total_uv  |
+-----------+
| 3         |
+-----------+
```

### 3.2 按日期分组统计

```sql
SELECT 
    date,
    hll_cardinality(hll_union_agg(user_id)) AS daily_uv
FROM user_visits
GROUP BY date;
```

返回结果示例：
```
+------------+-----------+
| date       | daily_uv  |
+------------+-----------+
| 2023-01-01 | 2         |
| 2023-01-02 | 2         |
+------------+-----------+
```

## 4. HLL函数详解

Doris提供了丰富的HLL函数，以下是常用函数对比：

```
| 函数名                | 描述                              | 示例                                |
|-----------------------|-----------------------------------|-------------------------------------|
| hll_hash(varchar)     | 将字符串转换为HLL值               | hll_hash('user123')                |
| hll_cardinality(hll)  | 估算HLL的基数                     | hll_cardinality(hll_column)        |
| hll_union(hll, hll)   | 合并两个HLL值                     | hll_union(hll1, hll2)              |
| hll_union_agg(hll)    | 聚合多行HLL值                     | hll_union_agg(hll_column)          |
| hll_empty()           | 创建一个空的HLL对象               | hll_empty()                        |
```

## 5. 实际应用示例

### 5.1 UV统计场景

```sql
-- 创建用户访问表
CREATE TABLE user_actions (
    dt DATE,
    page_id INT,
    user_hll HLL HLL_UNION
)
DISTRIBUTED BY HASH(dt, page_id) BUCKETS 16;

-- 插入数据
INSERT INTO user_actions VALUES 
('2023-01-01', 1, hll_hash('user1')),
('2023-01-01', 1, hll_hash('user2')),
('2023-01-01', 2, hll_hash('user1'));

-- 查询每个页面的UV
SELECT 
    dt,
    page_id,
    hll_cardinality(hll_union_agg(user_hll)) AS uv
FROM user_actions
GROUP BY dt, page_id;
```

返回结果示例：
```
+------------+---------+------+
| dt         | page_id | uv   |
+------------+---------+------+
| 2023-01-01 | 1       | 2    |
| 2023-01-01 | 2       | 1    |
+------------+---------+------+
```

### 5.2 留存率计算

```sql
-- 创建留存分析表
CREATE TABLE user_retention (
    first_date DATE,
    retention_date DATE,
    user_hll HLL HLL_UNION
)
DISTRIBUTED BY HASH(first_date) BUCKETS 10;

-- 计算7日留存率
SELECT
    first_date,
    hll_cardinality(hll_union_agg(CASE WHEN retention_date = first_date THEN user_hll ELSE hll_empty() END)) AS day1_uv,
    hll_cardinality(hll_union_agg(CASE WHEN retention_date = DATE_ADD(first_date, INTERVAL 7 DAY) THEN user_hll ELSE hll_empty() END)) AS day7_uv,
    hll_cardinality(hll_union_agg(CASE WHEN retention_date = DATE_ADD(first_date, INTERVAL 7 DAY) THEN user_hll ELSE hll_empty() END)) / 
    hll_cardinality(hll_union_agg(CASE WHEN retention_date = first_date THEN user_hll ELSE hll_empty() END)) AS retention_rate
FROM user_retention
GROUP BY first_date;
```

## 6. 注意事项

1. HLL是估算算法，结果不是精确值
2. 对于小基数（<100）的估算误差可能较大
3. HLL列不能直接用于WHERE条件过滤
4. 使用hll_hash函数时，相同输入总是产生相同输出

通过合理使用HLL功能，可以在Doris中高效实现大规模数据的基数统计，显著降低计算和存储资源消耗。