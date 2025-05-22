[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的 CTE (公共表表达式)

CTE (Common Table Expression，公共表表达式) 是 PostgreSQL 中一个非常有用的功能，它允许你创建临时结果集，该结果集仅存在于查询执行期间。

## 基本语法

```sql
WITH cte_name AS (
    SELECT ...  -- 定义CTE的查询
)
SELECT * FROM cte_name;  -- 使用CTE
```

## CTE 的特点

1. **临时性**：CTE 只在查询执行期间存在
2. **可读性**：可以简化复杂查询，提高可读性
3. **可递归**：PostgreSQL 支持递归 CTE

## 示例

### 简单 CTE

```sql
WITH regional_sales AS (
    SELECT region, SUM(amount) AS total_sales
    FROM orders
    GROUP BY region
)
SELECT region, total_sales
FROM regional_sales
WHERE total_sales > 1000000;
```

### 多个 CTE

```sql
WITH 
    sales AS (
        SELECT product_id, SUM(amount) AS total
        FROM orders
        GROUP BY product_id
    ),
    top_products AS (
        SELECT product_id
        FROM sales
        ORDER BY total DESC
        LIMIT 10
    )
SELECT p.name, s.total
FROM products p
JOIN sales s ON p.id = s.product_id
WHERE p.id IN (SELECT product_id FROM top_products);
```

### 递归 CTE

递归 CTE 常用于处理层次结构数据，如组织结构、树形结构等：

```sql
WITH RECURSIVE employee_hierarchy AS (
    -- 基础查询：选择顶级员工
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- 递归查询：选择下属员工
    SELECT e.id, e.name, e.manager_id, eh.level + 1
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.id
)
SELECT * FROM employee_hierarchy ORDER BY level, name;
```

## 使用场景

1. 简化复杂查询
2. 替代视图（当不想创建永久视图时）
3. 实现递归查询
4. 多次引用同一子查询结果

CTE 是 PostgreSQL 中组织复杂查询的强大工具，能显著提高查询的可读性和可维护性。