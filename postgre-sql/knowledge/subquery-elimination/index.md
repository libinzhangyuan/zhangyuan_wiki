[返回](/postgre-sql/knowledge/index)


# PostgreSQL 中的子查询消除（Subquery Elimination）

子查询消除是 PostgreSQL 查询优化器中的一项重要优化技术，它能够将某些类型的子查询转换为更高效的等价形式，从而提升查询性能。

## 什么是子查询消除？

子查询消除是指查询优化器在生成执行计划时，识别出可以重写或消除的子查询，将其转换为更简单的形式（如连接操作），或者完全消除子查询而不改变查询语义的过程。

## 子查询消除的类型

PostgreSQL 主要实现以下几种子查询消除优化：

### 1. 子查询提升（Subquery Pull-Up）

将 WHERE 子句或 FROM 子句中的相关子查询转换为连接操作。

**优化前：**
```sql
SELECT * FROM orders 
WHERE customer_id IN (SELECT id FROM customers WHERE status = 'active');
```

**优化后（等效形式）：**
```sql
SELECT orders.* FROM orders 
JOIN customers ON orders.customer_id = customers.id 
WHERE customers.status = 'active';
```

### 2. 子查询展开（Subquery Flattening）

将包含聚合函数的子查询展开为常规查询。

**优化前：**
```sql
SELECT name, (SELECT MAX(price) FROM products WHERE category = c.id) 
FROM categories c;
```

**优化后（等效形式）：**
```sql
SELECT c.name, MAX(p.price) 
FROM categories c LEFT JOIN products p ON p.category = c.id 
GROUP BY c.name;
```

### 3. EXISTS 子查询转换

将 EXISTS 子查询转换为半连接（SEMI JOIN）。

**优化前：**
```sql
SELECT * FROM products 
WHERE EXISTS (SELECT 1 FROM inventory WHERE inventory.product_id = products.id);
```

**优化后（等效形式）：**
```sql
SELECT products.* FROM products 
SEMI JOIN inventory ON inventory.product_id = products.id;
```

### 4. IN 子查询转换

将 IN 子查询转换为常规连接或哈希连接。

**优化前：**
```sql
SELECT * FROM employees 
WHERE department_id IN (SELECT id FROM departments WHERE location = 'NY');
```

**优化后（等效形式）：**
```sql
SELECT e.* FROM employees e 
JOIN departments d ON e.department_id = d.id 
WHERE d.location = 'NY';
```

## 子查询消除的优势

1. **性能提升**：连接操作通常比子查询执行效率更高
2. **更好的优化机会**：转换后的形式可能适用更多优化规则
3. **减少重复计算**：避免对子查询的重复执行
4. **更好的并行化**：连接操作比子查询更容易并行执行

## 限制条件

并非所有子查询都能被消除，以下情况通常难以优化：

1. 包含聚合函数、窗口函数或 LIMIT 的复杂子查询
2. 包含易变函数（volatile functions）的子查询
3. 某些类型的相关子查询
4. 包含 UNION/INTERSECT/EXCEPT 的子查询

## 查看优化效果

可以使用 `EXPLAIN` 命令查看 PostgreSQL 是否对子查询进行了消除：

```sql
EXPLAIN SELECT * FROM orders WHERE customer_id IN (SELECT id FROM customers);
```

如果看到执行计划中出现了连接操作而非子查询，说明子查询消除优化已经生效。

子查询消除是 PostgreSQL 查询优化器的重要组成部分，它能显著提高包含子查询的复杂查询的执行效率。