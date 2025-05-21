[返回](/postgre-sql/knowledge/index)


# PostgreSQL 中的 GROUP BY 子句

GROUP BY 是 PostgreSQL 中用于对查询结果进行分组的一个关键子句，它通常与聚合函数一起使用来对数据进行分组统计。

## 基本语法

```sql
SELECT column1, aggregate_function(column2)
FROM table_name
WHERE condition
GROUP BY column1;
```

## 主要功能

1. **数据分组**：将结果集按照一个或多个列的值进行分组
2. **配合聚合函数**：与 COUNT(), SUM(), AVG(), MAX(), MIN() 等聚合函数一起使用
3. **去重统计**：可以统计不同分组的数据情况

## 使用示例

### 简单分组

```sql
-- 按部门分组统计员工数量
SELECT department, COUNT(*) as employee_count
FROM employees
GROUP BY department;
```

### 多列分组

```sql
-- 按部门和职位分组统计
SELECT department, job_title, COUNT(*) 
FROM employees
GROUP BY department, job_title;
```

### 配合 HAVING 子句

```sql
-- 筛选出员工数量大于5的部门
SELECT department, COUNT(*) as emp_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
```

## 注意事项

1. SELECT 子句中非聚合函数的列必须出现在 GROUP BY 子句中
2. GROUP BY 可以基于表达式进行分组
3. 在 PostgreSQL 中，GROUP BY 子句不会对结果进行排序，如需排序需使用 ORDER BY
4. GROUP BY 可以与 DISTINCT 一起使用，但通常不需要，因为 GROUP BY 已经实现了去重

## 高级用法

PostgreSQL 还支持一些 GROUP BY 的高级功能：

- **GROUPING SETS**：一次查询中定义多个分组集
- **ROLLUP**：生成分层的小计和总计
- **CUBE**：生成所有可能的分组组合

这些高级分组功能使得复杂的数据分析查询更加高效和简洁。


# PostgreSQL 中的 GROUPING SETS

GROUPING SETS 是 PostgreSQL 中 GROUP BY 子句的一个高级功能，它允许在单个查询中指定多个分组集，相当于多个 GROUP BY 查询的 UNION ALL。

## 基本语法

```sql
SELECT column1, column2, aggregate_function(column3)
FROM table_name
GROUP BY GROUPING SETS (
    (column1, column2),
    (column1),
    (column2),
    ()
);
```

## 主要特点

1. **多维度分析**：可以一次性计算多个维度的聚合结果
2. **效率高**：比写多个 GROUP BY 查询并用 UNION ALL 连接更高效
3. **灵活性**：可以自由组合需要分析的分组维度

## 使用示例

### 基本示例

```sql
-- 同时按部门、按职位以及按部门和职位组合统计员工数量和平均工资
SELECT 
    department,
    job_title,
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY GROUPING SETS (
    (department, job_title),
    (department),
    (job_title),
    ()
);
```

### 实际应用

```sql
-- 销售数据多维度分析
SELECT 
    region,
    product_category,
    EXTRACT(YEAR FROM sale_date) AS year,
    SUM(amount) AS total_sales
FROM sales
GROUP BY GROUPING SETS (
    (region, product_category, year),  -- 最细粒度
    (region, year),                   -- 按地区和年份
    (product_category, year),         -- 按品类和年份
    (year),                           -- 仅按年份
    ()                                -- 总计
)
ORDER BY 
    CASE WHEN region IS NULL THEN 1 ELSE 0 END,
    region,
    CASE WHEN product_category IS NULL THEN 1 ELSE 0 END,
    product_category,
    year;
```

## 结果解释

使用 GROUPING SETS 时，结果集中会包含 NULL 值来表示聚合级别。例如：
- 当 region 为 NULL 时，表示该行是按其他列聚合的结果
- 当所有分组列都为 NULL 时，表示是总计行

## 与其它分组函数的比较

1. **GROUPING SETS**：明确指定需要哪些分组组合
2. **ROLLUP**：生成分层的小计和总计（特定类型的 GROUPING SETS）
3. **CUBE**：生成所有可能的分组组合（另一种特殊的 GROUPING SETS）

## 性能考虑

虽然 GROUPING SETS 很强大，但也要注意：
- 分组集越多，查询消耗的资源越大
- 只包含实际需要的分组组合
- 对于大数据集，可能需要优化查询计划

GROUPING SETS 是 PostgreSQL 中进行复杂数据分析的强大工具，特别适合需要从多个维度查看汇总数据的业务场景。