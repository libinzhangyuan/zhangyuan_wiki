[返回](/postgre-sql/knowledge/index)

# PostgreSQL 视图(VIEW)详解

视图是PostgreSQL中一个非常重要的数据库对象，它本质上是一个虚拟表，是基于一个或多个实际表的查询结果集。

## 视图的基本概念

视图不存储实际数据，而是存储查询定义。每次查询视图时，PostgreSQL都会执行视图定义中的查询语句。

### 创建视图的基本语法

```sql
CREATE VIEW view_name AS
SELECT column1, column2, ...
FROM table_name
WHERE condition;
```

## 视图示例

### 示例1：创建简单视图

假设我们有一个员工表`employees`：

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(50),
    salary NUMERIC(10,2),
    hire_date DATE
);

INSERT INTO employees (name, department, salary, hire_date) VALUES
('张三', '销售部', 8500, '2020-03-15'),
('李四', '技术部', 12000, '2019-07-22'),
('王五', '销售部', 7800, '2021-01-10'),
('赵六', '人事部', 9500, '2018-11-05'),
('钱七', '技术部', 13500, '2020-09-30');
```

创建一个只显示销售部员工的视图：

```sql
CREATE VIEW sales_employees AS
SELECT id, name, salary, hire_date
FROM employees
WHERE department = '销售部';
```

查询视图：

```sql
SELECT * FROM sales_employees;
```

```
 id | name | salary  | hire_date  
----+------+---------+------------
  1 | 张三 | 8500.00 | 2020-03-15
  3 | 王五 | 7800.00 | 2021-01-10
```

### 示例2：创建计算字段的视图

创建一个显示员工工龄和年薪的视图：

```sql
CREATE VIEW employee_stats AS
SELECT 
    id,
    name,
    department,
    salary,
    EXTRACT(YEAR FROM age(current_date, hire_date)) AS years_of_service,
    salary * 12 AS annual_salary
FROM employees;
```

查询视图：

```sql
SELECT * FROM employee_stats;
```

```
 id | name | department | salary  | years_of_service | annual_salary 
----+------+------------+---------+------------------+---------------
  1 | 张三 | 销售部     | 8500.00 |                3 |     102000.00
  2 | 李四 | 技术部     | 12000.00|                4 |     144000.00
  3 | 王五 | 销售部     | 7800.00 |                2 |      93600.00
  4 | 赵六 | 人事部     | 9500.00 |                5 |     114000.00
  5 | 钱七 | 技术部     | 13500.00|                3 |     162000.00
```

## 视图的优点

1. **简化复杂查询**：将复杂查询封装在视图中
2. **数据安全性**：可以隐藏敏感列或行
3. **逻辑独立性**：应用程序可以不受基础表结构变化的影响
4. **数据一致性**：确保所有用户看到相同的数据表示

## 修改视图

使用`CREATE OR REPLACE VIEW`语句修改视图定义：

```sql
CREATE OR REPLACE VIEW sales_employees AS
SELECT id, name, salary, hire_date, 
       EXTRACT(YEAR FROM age(current_date, hire_date)) AS years_of_service
FROM employees
WHERE department = '销售部';
```

## 删除视图

```sql
DROP VIEW IF EXISTS sales_employees;
```

## 物化视图(Materialized View)

PostgreSQL还支持物化视图，它会实际存储查询结果：

```sql
CREATE MATERIALIZED VIEW mv_employee_stats AS
SELECT department, AVG(salary) as avg_salary, COUNT(*) as employee_count
FROM employees
GROUP BY department;
```

刷新物化视图：

```sql
REFRESH MATERIALIZED VIEW mv_employee_stats;
```

查询物化视图：

```sql
SELECT * FROM mv_employee_stats;
```

```
 department |     avg_salary     | employee_count 
------------+--------------------+----------------
 销售部     |        8150.000000 |              2
 技术部     |       12750.000000 |              2
 人事部     |        9500.000000 |              1
```

## 视图与权限

可以为视图设置单独的权限，而不影响基础表：

```sql
GRANT SELECT ON sales_employees TO some_user;
REVOKE SELECT ON sales_employees FROM some_user;
```

视图是PostgreSQL中强大的功能，合理使用可以大大提高数据库的安全性和易用性。