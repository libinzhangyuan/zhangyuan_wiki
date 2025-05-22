[返回](/postgre-sql/knowledge/index)

# PostgreSQL 窗口函数(Window Function)详解

窗口函数是PostgreSQL中非常强大的功能，它允许你在不减少行数的情况下对数据集进行计算和分析。

## 基本概念

窗口函数与聚合函数类似，但不会将多行合并为一行，而是为每一行返回一个值。它通过`OVER()`子句定义"窗口"（即计算的行集合）。

## 基本语法

```sql
SELECT 
    列名,
    窗口函数() OVER(
        [PARTITION BY 分组列]
        [ORDER BY 排序列 [ASC|DESC]]
        [frame_clause]
    ) AS 别名
FROM 表名;
```

## 常用窗口函数

### 排名函数
- `ROW_NUMBER()` - 为每行分配唯一序号
- `RANK()` - 相同值有相同排名，后续排名跳过
- `DENSE_RANK()` - 相同值有相同排名，后续排名不跳过

### 分析函数
- `FIRST_VALUE()` - 返回窗口第一行的值
- `LAST_VALUE()` - 返回窗口最后一行的值
- `LAG()` - 访问当前行之前的行
- `LEAD()` - 访问当前行之后的行
- `NTILE(n)` - 将结果集分成n个桶

### 聚合函数作为窗口函数
- `SUM() OVER()`
- `AVG() OVER()`
- `COUNT() OVER()`
- `MAX() OVER()`
- `MIN() OVER()`

## 示例

### 基本使用
```sql
SELECT 
    department, 
    employee, 
    salary,
    AVG(salary) OVER(PARTITION BY department) AS avg_dept_salary
FROM employees;
```

### 排名示例
```sql
SELECT 
    product_id,
    sales,
    RANK() OVER(ORDER BY sales DESC) AS sales_rank
FROM products;
```

### 移动平均
```sql
SELECT 
    date,
    revenue,
    AVG(revenue) OVER(ORDER BY date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg
FROM daily_sales;
```

## 窗口框架(FRAME)

窗口框架定义了函数计算的具体范围：
- `ROWS BETWEEN ... AND ...`
- `RANGE BETWEEN ... AND ...`
- `GROUPS BETWEEN ... AND ...`

例如：
```sql
SUM(sales) OVER(ORDER BY date ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING)
```

窗口函数是数据分析的强大工具，特别适用于计算累计值、移动平均值、排名等场景。



# PostgreSQL 窗口函数返回结果示例

下面我将通过几个具体的SQL示例和它们的返回结果，展示窗口函数在实际查询中的效果。

## 示例1：基本窗口函数与部门平均工资

**测试数据 (employees 表):**
```
| id | department | employee | salary |
|----|------------|----------|--------|
| 1  | IT         | 张三     | 8000   |
| 2  | IT         | 李四     | 9000   |
| 3  | HR         | 王五     | 6000   |
| 4  | HR         | 赵六     | 6500   |
| 5  | Sales      | 钱七     | 7000   |
```

**SQL查询:**
```sql
SELECT 
    department, 
    employee, 
    salary,
    AVG(salary) OVER(PARTITION BY department) AS avg_dept_salary
FROM employees;
```

**返回结果:**
```
| department | employee | salary | avg_dept_salary |
|------------|----------|--------|-----------------|
| HR         | 王五     | 6000   | 6250            |
| HR         | 赵六     | 6500   | 6250            |
| IT         | 张三     | 8000   | 8500            |
| IT         | 李四     | 9000   | 8500            |
| Sales      | 钱七     | 7000   | 7000            |
```

## 示例2：排名函数

**SQL查询:**
```sql
SELECT 
    department, 
    employee, 
    salary,
    ROW_NUMBER() OVER(PARTITION BY department ORDER BY salary DESC) AS row_num,
    RANK() OVER(PARTITION BY department ORDER BY salary DESC) AS rank,
    DENSE_RANK() OVER(PARTITION BY department ORDER BY salary DESC) AS dense_rank
FROM employees;
```

**返回结果:**
```
| department | employee | salary | row_num | rank | dense_rank |
|------------|----------|--------|---------|------|------------|
| HR         | 赵六     | 6500   | 1       | 1    | 1          |
| HR         | 王五     | 6000   | 2       | 2    | 2          |
| IT         | 李四     | 9000   | 1       | 1    | 1          |
| IT         | 张三     | 8000   | 2       | 2    | 2          |
| Sales      | 钱七     | 7000   | 1       | 1    | 1          |
```

## 示例3：LAG和LEAD函数

**SQL查询:**
```sql
SELECT 
    employee,
    salary,
    LAG(salary, 1) OVER(ORDER BY salary) AS prev_salary,
    LEAD(salary, 1) OVER(ORDER BY salary) AS next_salary
FROM employees;
```

**返回结果:**
```
| employee | salary | prev_salary | next_salary |
|----------|--------|-------------|-------------|
| 王五     | 6000   | NULL        | 6500        |
| 赵六     | 6500   | 6000        | 7000        |
| 钱七     | 7000   | 6500        | 8000        |
| 张三     | 8000   | 7000        | 9000        |
| 李四     | 9000   | 8000        | NULL        |
```

## 示例4：累计求和

**SQL查询:**
```sql
SELECT 
    employee,
    salary,
    SUM(salary) OVER(ORDER BY salary ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM employees;
```

**返回结果:**
```
| employee | salary | running_total |
|----------|--------|---------------|
| 王五     | 6000   | 6000          |
| 赵六     | 6500   | 12500         |
| 钱七     | 7000   | 19500         |
| 张三     | 8000   | 27500         |
| 李四     | 9000   | 36500         |
```

## 示例5：NTILE分组

**SQL查询:**
```sql
SELECT 
    employee,
    salary,
    NTILE(3) OVER(ORDER BY salary) AS salary_group
FROM employees;
```

**返回结果:**
```
| employee | salary | salary_group |
|----------|--------|--------------|
| 王五     | 6000   | 1            |
| 赵六     | 6500   | 1            |
| 钱七     | 7000   | 2            |
| 张三     | 8000   | 2            |
| 李四     | 9000   | 3            |
```

这些示例展示了窗口函数如何在不减少原始行数的情况下，为每一行添加计算后的值。窗口函数在实际数据分析中非常有用，特别是当需要比较单行与组内或整体数据的关系时。