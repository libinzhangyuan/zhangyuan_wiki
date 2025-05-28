[返回](/postgre-sql/knowledge/index)


# PostgreSQL 的 Row Value Syntax 介绍

Row Value Syntax（行值语法）是 PostgreSQL 中一种强大的语法特性，它允许你将多个值组合成一个逻辑单元（行构造器）来处理，这在比较、子查询和批量操作中特别有用。

## 基本语法

行值语法使用括号将多个值组合在一起：

```sql
(值1, 值2, 值3, ...)
```

## 主要用途

### 1. 多列比较

可以同时比较多个列：

```sql
SELECT * FROM employees 
WHERE (first_name, last_name) = ('John', 'Doe');
```

**返回示例：**
```
| employee_id | first_name | last_name | department | salary |
|-------------|------------|-----------|------------|--------|
| 101         | John       | Doe       | Sales      | 50000  |
```
### 2. IN 操作符中使用行值

```sql
SELECT * FROM products 
WHERE (category, price) IN (('Electronics', 999), ('Clothing', 49));
```

**返回示例：**
```
| product_id | product_name | category    | price |
|------------|--------------|-------------|-------|
| 1          | Smartphone   | Electronics | 999   |
| 5          | T-Shirt      | Clothing    | 49    |
```
### 3. BETWEEN 操作符中使用行值

```sql
SELECT * FROM orders 
WHERE (order_date, total_amount) 
BETWEEN ('2023-01-01', 100) AND ('2023-01-31', 500);
```

**返回示例：**
```
| order_id | customer_id | order_date | total_amount |
|----------|-------------|------------|--------------|
| 1001     | 201         | 2023-01-15 | 299.99       |
| 1002     | 205         | 2023-01-20 | 150.50       |
```
### 4. 行值构造函数

可以直接构造行值：

```sql
SELECT ROW(1, 'apple', 2.99) AS product_info;
```

**返回示例：**
```
| product_info     |
|------------------|
| (1,apple,2.99)   |
```
### 5. 子查询中使用行值

```sql
SELECT * FROM students 
WHERE (grade, score) = (SELECT grade, MAX(score) 
                        FROM students 
                        GROUP BY grade 
                        HAVING grade = 'A');
```

**返回示例：**
```
| student_id | student_name | grade | score |
|------------|--------------|-------|-------|
| 101        | Alice        | A     | 98    |
| 105        | Bob          | A     | 98    |
```
### 6. 更新多列

```sql
UPDATE employees 
SET (salary, department) = (SELECT avg_salary, 'Management' 
                            FROM (SELECT AVG(salary) AS avg_salary 
                                  FROM employees) AS subq) 
WHERE employee_id = 101;
```

**更新后查询示例：**

```sql
SELECT * FROM employees WHERE employee_id = 101;
```

**返回示例：**
```
| employee_id | first_name | last_name | department | salary  |
|-------------|------------|-----------|------------|---------|
| 101         | John       | Doe       | Management | 75000.0 |
```
## 高级用法

### 行值比较

PostgreSQL 支持逐元素比较行值：

```sql
SELECT (1, 'a') < (2, 'b') AS is_less;
```

**返回示例：**
```
| is_less |
|---------|
| true    |
```
### 与 JOIN 结合使用

```sql
SELECT e.* 
FROM employees e
JOIN (VALUES (1, 'Sales'), (2, 'Marketing')) AS dept(id, name)
ON (e.department_id, e.department) = (dept.id, dept.name);
```

**返回示例：**
```
| employee_id | first_name | last_name | department_id | department | salary |
|-------------|------------|-----------|---------------|------------|--------|
| 101         | John       | Doe       | 1             | Sales      | 50000  |
| 102         | Jane       | Smith     | 2             | Marketing  | 60000  |
```
## 注意事项

1. 行值语法中的 NULL 处理需要特别注意，因为 `(NULL = NULL)` 的结果是 NULL 而不是 true
2. 行值比较是按顺序逐个字段比较的
3. 行值语法可以大大提高某些复杂查询的可读性和性能

行值语法是 PostgreSQL 中一个强大但常被忽视的特性，合理使用可以使你的 SQL 查询更加简洁和高效。