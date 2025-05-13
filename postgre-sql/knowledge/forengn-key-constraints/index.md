[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的外键约束 (Foreign Key Constraints)

外键约束是关系数据库中的一种重要机制，用于维护表之间的引用完整性。在 PostgreSQL 中，外键约束确保一个表（子表）中的列值必须匹配另一个表（父表）中的现有值。

## 基本概念

外键约束定义了表之间的关系：
- **父表 (被引用表)**: 包含被引用的主键或唯一键
- **子表 (引用表)**: 包含引用父表键的外键列

## 创建外键约束

### 1. 创建表时定义外键

```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    order_date DATE NOT NULL,
    -- 定义外键约束
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

### 2. 为现有表添加外键

```sql
ALTER TABLE orders
ADD CONSTRAINT fk_orders_customers
FOREIGN KEY (customer_id) REFERENCES customers(customer_id);
```

## 外键约束选项

### ON DELETE 行为

指定当父表记录被删除时的行为：

```sql
FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
ON DELETE CASCADE;  -- 级联删除子表记录

-- 其他选项：
-- ON DELETE RESTRICT (默认) - 阻止删除
-- ON DELETE SET NULL - 将外键设为NULL
-- ON DELETE SET DEFAULT - 将外键设为默认值
-- ON DELETE NO ACTION - 类似于RESTRICT
```

### ON UPDATE 行为

指定当父表键值更新时的行为：

```sql
FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
ON UPDATE CASCADE;  -- 级联更新子表外键值

-- 其他选项与ON DELETE类似
```

## 多列外键

可以创建基于多列的外键约束：

```sql
CREATE TABLE order_items (
    order_id INTEGER,
    item_id INTEGER,
    quantity INTEGER,
    PRIMARY KEY (order_id, item_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (item_id) REFERENCES items(item_id)
);

-- 或者组合外键
CREATE TABLE employee_projects (
    employee_id INTEGER,
    project_id INTEGER,
    role VARCHAR(50),
    PRIMARY KEY (employee_id, project_id),
    FOREIGN KEY (employee_id, project_id) 
    REFERENCES project_assignments(employee_id, project_id)
);
```

## 查看外键约束

### 1. 使用 `\d` 命令（在psql中）

```sql
\d table_name
```

### 2. 查询系统目录

```sql
SELECT
    tc.constraint_name,
    tc.table_name,
    kcu.column_name,
    ccu.table_name AS foreign_table_name,
    ccu.column_name AS foreign_column_name
FROM
    information_schema.table_constraints AS tc
    JOIN information_schema.key_column_usage AS kcu
      ON tc.constraint_name = kcu.constraint_name
    JOIN information_schema.constraint_column_usage AS ccu
      ON ccu.constraint_name = tc.constraint_name
WHERE
    tc.constraint_type = 'FOREIGN KEY' AND
    tc.table_name = 'your_table_name';
```

## 删除外键约束

```sql
ALTER TABLE table_name
DROP CONSTRAINT constraint_name;
```

## 外键约束的注意事项

1. **性能影响**:
   - 外键会带来一定的性能开销，特别是在大量插入/更新操作时
   - 但能避免数据不一致带来的更大问题

2. **锁行为**:
   - 外键操作可能获取父表的锁，可能引起死锁
   - 在事务中操作多个表时要小心顺序

3. **延迟约束检查**:
   ```sql
   -- 可以在事务中延迟约束检查
   ALTER TABLE table_name
   ADD CONSTRAINT constraint_name
   FOREIGN KEY (column) REFERENCES other_table(column)
   DEFERRABLE INITIALLY DEFERRED;
   ```

4. **索引建议**:
   - 外键列通常应该建立索引以提高连接性能
   - PostgreSQL 不会自动为外键列创建索引

5. **循环引用**:
   - 两个或多个表相互引用可能造成问题
   - 可以使用 DEFERRABLE 约束或临时禁用约束

外键约束是维护数据完整性的强大工具，合理使用可以确保数据库中的数据关系始终保持一致。