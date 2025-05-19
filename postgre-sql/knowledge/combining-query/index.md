[返回](/postgre-sql/knowledge/index)


# PostgreSQL 中的 Combining Query（组合查询）详解

在 PostgreSQL 中，Combining Query（组合查询）指的是将多个 SELECT 语句的结果合并成一个结果集的操作。这种功能允许您从不同的查询中获取数据并以统一的方式处理它们。

## 主要的组合查询操作符

PostgreSQL 提供了三种主要的组合查询操作符：

### 1. UNION（并集）

**功能**：合并两个或多个 SELECT 语句的结果集，并自动去除重复行。

**语法**：
```sql
SELECT column1, column2, ... FROM table1
UNION
SELECT column1, column2, ... FROM table2;
```

**特点**：
- 两个 SELECT 语句必须有相同数量的列
- 对应列的数据类型必须兼容
- 结果集中的列名取自第一个 SELECT 语句
- 默认会去除重复行（如果需要保留重复行，使用 UNION ALL）

**示例**：
```sql
-- 获取所有员工和客户的姓名
SELECT name FROM employees
UNION
SELECT name FROM customers;
```

### 2. INTERSECT（交集）

**功能**：返回两个 SELECT 语句结果集的共同部分（即同时存在于两个结果集中的行）。

**语法**：
```sql
SELECT column1, column2, ... FROM table1
INTERSECT
SELECT column1, column2, ... FROM table2;
```

**特点**：
- 同样要求 SELECT 语句有相同数量的列且类型兼容
- 结果只包含在两个查询中都存在的行
- 默认去除重复行

**示例**：
```sql
-- 找出既是员工又是客户的人
SELECT name FROM employees
INTERSECT
SELECT name FROM customers;
```

### 3. EXCEPT（差集）

**功能**：返回第一个 SELECT 语句结果集中不包含在第二个 SELECT 语句结果集中的行。

**语法**：
```sql
SELECT column1, column2, ... FROM table1
EXCEPT
SELECT column1, column2, ... FROM table2;
```

**特点**：
- 返回只在第一个查询中存在而不在第二个查询中的行
- 在某些数据库系统中也称为 MINUS

**示例**：
```sql
-- 找出是员工但不是客户的人
SELECT name FROM employees
EXCEPT
SELECT name FROM customers;
```

## 组合查询的高级用法

### 使用 UNION ALL 保留重复行

```sql
-- 获取所有订单ID，包括重复的
SELECT order_id FROM online_orders
UNION ALL
SELECT order_id FROM store_orders;
```

### 组合多个查询

```sql
SELECT a FROM t1
UNION
SELECT b FROM t2
INTERSECT
SELECT c FROM t3;
```

### 带排序的组合查询

```sql
(SELECT name, salary FROM employees WHERE department = 'IT')
UNION
(SELECT name, salary FROM employees WHERE department = 'HR')
ORDER BY salary DESC;
```

### 与 GROUP BY 和聚合函数结合

```sql
(SELECT department, COUNT(*) as count FROM employees GROUP BY department)
UNION
(SELECT 'Total' as department, COUNT(*) as count FROM employees)
ORDER BY count DESC;
```

## 性能考虑

1. **UNION vs UNION ALL**：如果确定没有重复行或不需要去重，使用 UNION ALL 更高效，因为它不需要额外的去重步骤。

2. **索引利用**：组合查询中的每个 SELECT 语句都可以利用各自的索引。

3. **执行顺序**：PostgreSQL 会先执行各个 SELECT 语句，然后再应用组合操作。

4. **LIMIT 使用**：可以在组合查询后使用 LIMIT，也可以在各个 SELECT 语句中使用，但效果不同。

## 实际应用场景

1. **数据分片查询**：从多个结构相同的表中查询数据
2. **历史数据和当前数据合并**：查询当前表和历史归档表
3. **多条件筛选**：用 UNION 实现 OR 逻辑，有时比 OR 条件更高效
4. **数据对比**：使用 INTERSECT 和 EXCEPT 比较数据集差异
5. **报表生成**：合并多个维度的统计结果

组合查询是 PostgreSQL 中强大的功能，合理使用可以简化复杂的数据操作，但也要注意性能影响，特别是在处理大数据集时。