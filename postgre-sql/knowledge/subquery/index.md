[返回](/postgre-sql/knowledge/index)

# PostgreSQL 子查询(Subqueries)详解

子查询是指嵌套在另一个SQL查询中的SELECT语句，在PostgreSQL中也被称为内部查询或嵌套查询。子查询是构建复杂SQL查询的强大工具。

## 子查询的主要类型

### 1. 标量子查询(Scalar Subquery)
- 返回单个值(一行一列)
- 可以用在SELECT、WHERE、HAVING等子句中

```sql
SELECT name, (SELECT AVG(price) FROM products) AS avg_price
FROM products;
```

### 2. 行子查询(Row Subquery)
- 返回单行多列
- 通常与行比较运算符一起使用

```sql
SELECT * FROM employees
WHERE (department, salary) = (SELECT department, MAX(salary) 
                             FROM employees GROUP BY department LIMIT 1);
```

### 3. 列子查询(Column Subquery)
- 返回单列多行
- 常与IN、ANY/SOME、ALL等运算符一起使用

```sql
SELECT name FROM products
WHERE id IN (SELECT product_id FROM orders WHERE order_date > '2023-01-01');
```

### 4. 表子查询(Table Subquery)
- 返回多行多列
- 通常用在FROM子句中

```sql
SELECT d.department_name, e.avg_salary
FROM departments d
JOIN (SELECT department_id, AVG(salary) AS avg_salary 
      FROM employees GROUP BY department_id) e
ON d.id = e.department_id;
```

## 子查询的使用位置

### WHERE子句中的子查询
```sql
SELECT name FROM products
WHERE price > (SELECT AVG(price) FROM products);
```

### FROM子句中的子查询(派生表)
```sql
SELECT dept.name, emp.employee_count
FROM departments dept
JOIN (SELECT department_id, COUNT(*) AS employee_count
      FROM employees GROUP BY department_id) emp
ON dept.id = emp.department_id;
```

### SELECT子句中的子查询
```sql
SELECT name, 
       (SELECT COUNT(*) FROM orders WHERE orders.product_id = products.id) AS order_count
FROM products;
```

### HAVING子句中的子查询
```sql
SELECT department_id, AVG(salary)
FROM employees
GROUP BY department_id
HAVING AVG(salary) > (SELECT AVG(salary) FROM employees);
```

## 子查询相关运算符

- **IN/NOT IN**：检查值是否在子查询结果中
- **EXISTS/NOT EXISTS**：检查子查询是否返回任何行
- **ANY/SOME**：与比较运算符一起使用，满足任一条件
- **ALL**：与比较运算符一起使用，满足所有条件

## 性能考虑

1. 相关子查询(引用外部查询列)通常比非相关子查询性能差
2. 考虑使用JOIN重写某些子查询以提高性能
3. 对大型数据集，EXISTS通常比IN性能更好

PostgreSQL优化器能有效处理许多子查询，但在复杂情况下可能需要手动优化。