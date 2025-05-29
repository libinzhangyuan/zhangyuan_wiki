[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的 UPSERT 操作详解

UPSERT 是 UPDATE 和 INSERT 的组合操作，表示"如果记录存在则更新，不存在则插入"。PostgreSQL 从 9.5 版本开始支持这种操作。

## 基本语法

PostgreSQL 使用 `INSERT ... ON CONFLICT` 语法实现 UPSERT：

```sql
INSERT INTO table_name (column_list) 
VALUES (value_list)
ON CONFLICT (conflict_target) 
DO UPDATE SET column1 = value1, column2 = value2, ...;
```

或者如果冲突时不想做任何操作：

```sql
INSERT INTO table_name (column_list) 
VALUES (value_list)
ON CONFLICT (conflict_target) 
DO NOTHING;
```

## 关键组成部分

1. **conflict_target** - 指定可能引发冲突的列或约束
   - 可以是主键：`ON CONFLICT (id)`
   - 可以是唯一约束：`ON CONFLICT ON CONSTRAINT constraint_name`
   - 可以是多列：`ON CONFLICT (col1, col2)`

2. **DO UPDATE** - 冲突时要执行的更新操作
3. **DO NOTHING** - 冲突时不执行任何操作

## 使用示例

### 示例1：基本 UPSERT 操作

假设有一个用户表，id 是主键：

```sql
INSERT INTO users (id, name, email, last_login) 
VALUES (1, '张三', 'zhangsan@example.com', '2023-01-01')
ON CONFLICT (id) 
DO UPDATE SET 
    name = EXCLUDED.name,
    email = EXCLUDED.email,
    last_login = EXCLUDED.last_login;
```

`EXCLUDED` 是一个特殊的表，包含要插入的行数据。

### 示例2：只更新特定字段

```sql
INSERT INTO products (id, name, price, stock) 
VALUES (101, '笔记本电脑', 5999, 10)
ON CONFLICT (id) 
DO UPDATE SET 
    stock = products.stock + EXCLUDED.stock;
```

这样如果产品已存在，只增加库存量而不改变价格。

### 示例3：DO NOTHING 示例

```sql
INSERT INTO categories (id, name) 
VALUES (5, '电子产品')
ON CONFLICT (id) 
DO NOTHING;
```

如果 id=5 的类别已存在，则不执行任何操作。

### 示例4：使用唯一约束作为冲突目标

```sql
INSERT INTO employees (employee_id, email, name) 
VALUES (123, 'lisi@company.com', '李四')
ON CONFLICT ON CONSTRAINT employees_email_key 
DO UPDATE SET name = EXCLUDED.name;
```

### 示例5：多列冲突目标

```sql
INSERT INTO bookings (room_id, booking_date, guest_name) 
VALUES (101, '2023-12-25', '王五')
ON CONFLICT (room_id, booking_date) 
DO UPDATE SET guest_name = EXCLUDED.guest_name;
```

## 特殊用法

### 条件更新

```sql
INSERT INTO logs (id, message, count) 
VALUES (1, '系统启动', 1)
ON CONFLICT (id) 
DO UPDATE SET 
    count = logs.count + 1,
    message = EXCLUDED.message
WHERE logs.count < 10;
```

### 使用 WHERE 子句限制冲突

```sql
INSERT INTO orders (order_id, status, customer_id) 
VALUES (1001, 'pending', 5)
ON CONFLICT (order_id) WHERE (status = 'pending') 
DO UPDATE SET 
    customer_id = EXCLUDED.customer_id;
```

## 注意事项

1. **冲突目标必须**：是唯一约束或主键
2. **性能考虑**：UPSERT 比单独的 INSERT 或 UPDATE 开销更大
3. **并发问题**：虽然 UPSERT 是原子操作，但在高并发环境下仍需注意
4. **返回结果**：可以使用 `RETURNING` 子句获取操作后的数据

```sql
INSERT INTO users (id, name) 
VALUES (1, '张三')
ON CONFLICT (id) 
DO UPDATE SET name = EXCLUDED.name
RETURNING id, name, (xmax = 0) AS inserted;
```

## 与传统方法的比较

在 PostgreSQL 9.5 之前，实现 UPSERT 需要使用以下模式：

```sql
-- 方法1：先尝试更新，如果没更新到再插入
UPDATE table SET ... WHERE id = X;
IF NOT FOUND THEN
    INSERT INTO table ...;
END IF;

-- 方法2：使用 WITH 子句
WITH upsert AS (
    UPDATE table SET ... WHERE id = X RETURNING *
)
INSERT INTO table (...) SELECT ... WHERE NOT EXISTS (SELECT 1 FROM upsert);
```

新的 `INSERT ... ON CONFLICT` 语法更简洁且性能更好。