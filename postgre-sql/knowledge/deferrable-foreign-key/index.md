[返回](/postgre-sql/knowledge/index)


# PostgreSQL 中的可延迟外键(Deferrable Foreign Keys)

## 基本概念

### 什么是可延迟约束？
- 在 PostgreSQL 中，外键约束通常会在每条 SQL 语句执行后立即检查
- 可延迟约束允许你将这种检查推迟到事务结束时进行

### 两种可延迟模式
1. **`DEFERRABLE INITIALLY IMMEDIATE`** (默认延迟模式)
   - 约束在每条语句后立即检查
   - 但可以通过 `SET CONSTRAINTS` 命令临时改为延迟检查

2. **`DEFERRABLE INITIALLY DEFERRED`** (初始延迟)
   - 约束默认在事务结束时才检查
   - 也可以通过 `SET CONSTRAINTS` 改为立即检查

## 创建可延迟外键

```sql
-- 创建表时指定可延迟外键
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id) 
        DEFERRABLE INITIALLY DEFERRED,
    order_date DATE
);

-- 修改现有约束为可延迟
ALTER TABLE orders 
    ALTER CONSTRAINT orders_customer_id_fkey 
    DEFERRABLE INITIALLY DEFERRED;
```

## 使用场景

### 典型用例
1. **循环引用**：当两个表互相引用时
   ```sql
   CREATE TABLE person (
       id SERIAL PRIMARY KEY,
       spouse_id INT,
       FOREIGN KEY (spouse_id) REFERENCES person(id)
           DEFERRABLE INITIALLY DEFERRED
   );
   ```

2. **批量数据加载**：需要先插入父表再更新子表引用时

3. **复杂事务**：需要暂时违反约束但最终会满足的情况

## 事务中控制约束检查

```sql
BEGIN;
-- 将约束设置为延迟检查
SET CONSTRAINTS orders_customer_id_fkey DEFERRED;

-- 现在可以自由操作数据，即使暂时违反外键约束
INSERT INTO orders (customer_id, order_date) VALUES (999, '2023-01-01');
UPDATE customers SET customer_id = 999 WHERE customer_id = 1;

-- 在事务提交时才会检查约束
COMMIT;
```

## 注意事项

1. 性能影响：延迟检查可能增加事务失败的风险
2. 不是所有约束都支持延迟（如 EXCLUDE 约束不支持）
3. 谨慎使用，确保事务结束时数据一致
4. 在并发环境中要注意可能出现的死锁情况

通过合理使用可延迟外键，可以更灵活地处理复杂的数据库操作场景。