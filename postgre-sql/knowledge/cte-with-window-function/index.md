[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中结合 CTE 和窗口函数的使用

CTE (公共表表达式) 和窗口函数是 PostgreSQL 中两个强大的功能，结合使用可以解决许多复杂的数据分析问题。

## 基本概念

### 窗口函数
```
窗口函数在对一组行(称为窗口)进行计算时，不会将这些行合并为单个输出行，而是保留所有行的原始形态。常用窗口函数包括：
- `ROW_NUMBER()` - 为行分配唯一序号
- `RANK()` - 为行分配排名，相同值有相同排名，会有间隔
- `DENSE_RANK()` - 类似 RANK() 但无间隔
- `LEAD()/LAG()` - 访问前后行的值
- `SUM()/AVG()/COUNT()` 等聚合函数的窗口版本
```
## CTE 与窗口函数结合示例

### 示例1：计算销售排名

```sql
WITH sales_data AS (
    SELECT 
        salesperson_id,
        SUM(amount) AS total_sales
    FROM sales
    GROUP BY salesperson_id
),
ranked_sales AS (
    SELECT 
        salesperson_id,
        total_sales,
        RANK() OVER (ORDER BY total_sales DESC) AS sales_rank,
        DENSE_RANK() OVER (ORDER BY total_sales DESC) AS dense_sales_rank,
        ROW_NUMBER() OVER (ORDER BY total_sales DESC) AS row_num
    FROM sales_data
)
SELECT * FROM ranked_sales WHERE sales_rank <= 10;
```

### 示例2：计算移动平均

```sql
WITH daily_sales AS (
    SELECT 
        sale_date,
        SUM(amount) AS daily_total
    FROM sales
    GROUP BY sale_date
)
SELECT 
    sale_date,
    daily_total,
    AVG(daily_total) OVER (ORDER BY sale_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS weekly_avg,
    AVG(daily_total) OVER (ORDER BY sale_date ROWS BETWEEN 29 PRECEDING AND CURRENT ROW) AS monthly_avg
FROM daily_sales
ORDER BY sale_date;
```

### 示例3：计算同比环比增长率

```sql
WITH monthly_sales AS (
    SELECT 
        EXTRACT(YEAR FROM sale_date) AS year,
        EXTRACT(MONTH FROM sale_date) AS month,
        SUM(amount) AS monthly_total
    FROM sales
    GROUP BY year, month
),
sales_with_lag AS (
    SELECT 
        year,
        month,
        monthly_total,
        LAG(monthly_total, 1) OVER (ORDER BY year, month) AS prev_month,
        LAG(monthly_total, 12) OVER (ORDER BY year, month) AS prev_year_month
    FROM monthly_sales
)
SELECT 
    year,
    month,
    monthly_total,
    ROUND((monthly_total - prev_month) / prev_month * 100, 2) AS mom_growth,
    ROUND((monthly_total - prev_year_month) / prev_year_month * 100, 2) AS yoy_growth
FROM sales_with_lag
ORDER BY year, month;
```

## 高级用法

### 分区计算 (PARTITION BY)

```sql
WITH department_sales AS (
    SELECT 
        department_id,
        employee_id,
        SUM(amount) AS total_sales
    FROM sales
    GROUP BY department_id, employee_id
)
SELECT 
    department_id,
    employee_id,
    total_sales,
    RANK() OVER (PARTITION BY department_id ORDER BY total_sales DESC) AS dept_rank,
    total_sales / SUM(total_sales) OVER (PARTITION BY department_id) * 100 AS dept_percentage
FROM department_sales;
```

### 自定义窗口框架

```sql
WITH monthly_revenue AS (
    SELECT 
        date_trunc('month', order_date) AS month,
        SUM(revenue) AS revenue
    FROM orders
    GROUP BY month
)
SELECT 
    month,
    revenue,
    SUM(revenue) OVER (ORDER BY month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS trailing_3month,
    AVG(revenue) OVER (ORDER BY month ROWS BETWEEN 11 PRECEDING AND CURRENT ROW) AS trailing_12month_avg
FROM monthly_revenue
ORDER BY month;
```

## 性能考虑

1. 窗口函数在 CTE 中计算后，后续查询可以直接使用结果
2. 对于大数据集，考虑添加适当的索引
3. 复杂的窗口函数可能会影响性能，应合理使用

这种组合特别适合需要多层次数据分析和转换的场景，能够保持查询逻辑清晰同时处理复杂计算。