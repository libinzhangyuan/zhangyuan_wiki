[fanhui](/postgre-sql/knowledge/index)


# PostgreSQL 生成列（Generated Column）详解

生成列（Generated Column）是 PostgreSQL 12 引入的特性，它允许你创建其值自动根据其他列计算得出的列，这些列的值不能被直接插入或更新，只能通过定义的生成表达式自动计算。

## 基本语法

```sql
CREATE TABLE table_name (
    column_name data_type 
    GENERATED ALWAYS AS (generation_expression) STORED
);
```

## 生成列类型

PostgreSQL 支持两种生成列：

1. **STORED（存储型）**：值被实际存储在表中，占用存储空间
2. **VIRTUAL（虚拟型）**：PostgreSQL 目前不支持纯虚拟列（但可以用视图模拟）

## 实际示例

### 示例1：基本使用

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    price NUMERIC(10,2),
    quantity INTEGER,
    total_price NUMERIC(10,2) GENERATED ALWAYS AS (price * quantity) STORED
);

INSERT INTO products (price, quantity) VALUES (19.99, 5);
SELECT * FROM products;
```

**返回结果**：
```
| id | price | quantity | total_price |
|----|-------|----------|-------------|
| 1  | 19.99 | 5        | 99.95       |
```

### 示例2：使用字符串操作

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    first_name TEXT,
    last_name TEXT,
    full_name TEXT GENERATED ALWAYS AS (first_name || ' ' || last_name) STORED
);

INSERT INTO employees (first_name, last_name) VALUES ('张', '三');
SELECT * FROM employees;
```

**返回结果**：
```
| id | first_name | last_name | full_name |
|----|------------|-----------|-----------|
| 1  | 张         | 三        | 张 三     |
```

### 示例3：使用JSON数据

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    order_data JSONB,
    order_date DATE GENERATED ALWAYS AS ((order_data->>'order_date')::DATE) STORED
);

INSERT INTO orders (order_data) 
VALUES ('{"order_id": 1001, "order_date": "2023-05-15", "amount": 150.00}');
SELECT * FROM orders;
```

**返回结果**：
```
| id | order_data                                      | order_date |
|----|-------------------------------------------------|------------|
| 1  | {"amount": 150.00, "order_id": 1001, "order_date": "2023-05-15"} | 2023-05-15 |
```

## 重要注意事项

1. **不可直接更新**：尝试更新生成列会导致错误
   ```sql
   UPDATE products SET total_price = 100;  -- 错误！
   ```

2. **表达式限制**：生成表达式只能使用不可变函数（IMMUTABLE）

3. **性能考虑**：
   - STORED 列会占用存储空间但读取快
   - 对于复杂计算，考虑使用视图或触发器

4. **依赖关系**：生成列依赖于其他列，修改这些列会自动更新生成列

## 与视图的比较
```
| 特性                | 生成列             | 视图               |
|---------------------|--------------------|--------------------|
| 存储方式            | 实际存储（STORED） | 不存储，实时计算   |
| 写入性能            | 插入时计算         | 不影响写入         |
| 读取性能            | 更快               | 取决于查询复杂度   |
| 更新机制            | 自动               | 总是最新           |
| 索引支持            | 支持               | 通过物化视图支持   |
```

## 实际应用场景

1. **数据完整性**：确保计算列始终与源数据同步
2. **简化查询**：避免在多个查询中重复复杂表达式
3. **数据转换**：自动格式化或转换存储的数据
4. **业务逻辑**：实现简单的业务规则计算

生成列是PostgreSQL中提高数据一致性和简化应用逻辑的强大工具，特别适合那些需要基于其他列自动计算值的场景。