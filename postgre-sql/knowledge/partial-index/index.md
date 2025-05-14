[返回](/postgre-sql/knowledge/index)

# PostgreSQL 部分索引（Partial Index）详解

部分索引（Partial Index）是 PostgreSQL 中一种强大的索引类型，它只对表中满足特定条件的行建立索引，而不是为所有行建立索引。

## 基本概念

部分索引是通过在 `CREATE INDEX` 语句中添加 `WHERE` 子句创建的：

```sql
CREATE INDEX index_name 
ON table_name (column_name)
WHERE condition;
```

## 主要优势

1. **减小索引大小**：只索引需要的行
2. **提高查询性能**：更小的索引更高效
3. **减少维护开销**：DML操作只需更新部分索引
4. **实现特殊约束**：如唯一性约束只应用于数据子集

## 常见使用场景

### 1. 索引热点数据

```sql
-- 只为活跃用户创建索引
CREATE INDEX idx_active_users_email 
ON users (email) 
WHERE is_active = true;
```

### 2. 实现条件唯一约束

```sql
-- 确保每个用户只有一个未完成的订单
CREATE UNIQUE INDEX idx_one_pending_order_per_user 
ON orders (user_id) 
WHERE status = 'pending';
```

### 3. 排除NULL值

```sql
-- 只为非NULL值创建索引
CREATE INDEX idx_products_not_null_price 
ON products (price) 
WHERE price IS NOT NULL;
```

### 4. 时间范围索引

```sql
-- 只为最近的数据创建索引
CREATE INDEX idx_recent_logs 
ON access_logs (log_time) 
WHERE log_time > CURRENT_DATE - INTERVAL '30 days';
```

## 实际案例

### 案例1：电商平台订单索引

```sql
-- 只为未支付订单创建索引
CREATE INDEX idx_unpaid_orders 
ON orders (user_id, created_at) 
WHERE payment_status = 'unpaid';

-- 此查询将使用部分索引
EXPLAIN SELECT * FROM orders 
WHERE user_id = 1001 AND payment_status = 'unpaid';
```

### 案例2：社交平台消息索引

```sql
-- 只为未读消息创建索引
CREATE INDEX idx_unread_messages 
ON messages (receiver_id, created_at) 
WHERE is_read = false;

-- 此查询将使用部分索引
EXPLAIN SELECT * FROM messages 
WHERE receiver_id = 2002 AND is_read = false;
```

## 性能对比

假设表有100万行数据，其中10%满足条件：

| 索引类型 | 索引大小 | 查询性能 | 维护开销 |
|---------|---------|---------|---------|
| 完整索引 | 100MB   | 快      | 高      |
| 部分索引 | 10MB    | 更快    | 低      |

## 创建语法细节

```sql
CREATE [ UNIQUE ] INDEX [ CONCURRENTLY ] [ [ IF NOT EXISTS ] ] index_name
    ON [ ONLY ] table_name [ USING method ]
    ( { column_name | ( expression ) } [ COLLATE collation ] [ opclass ] [ ASC | DESC ] [ NULLS { FIRST | LAST } ] [, ...] )
    [ INCLUDE ( column_name [, ...] ) ]
    [ WITH ( storage_parameter = value [, ... ] ) ]
    [ TABLESPACE tablespace_name ]
    [ WHERE predicate ]  -- 这就是部分索引的关键
```

## 使用注意事项

1. **查询条件必须匹配**：查询的WHERE条件必须与索引的WHERE条件兼容才能使用部分索引
   
2. **计划器选择**：即使查询匹配，优化器也可能选择不使用部分索引

3. **统计信息**：确保ANALYZE及时执行，以便优化器了解部分索引的选择性

4. **表达式索引**：可以与部分索引结合使用

   ```sql
   CREATE INDEX idx_lower_name_active 
   ON users (LOWER(name)) 
   WHERE is_active = true;
   ```

## 查看部分索引

```sql
SELECT indexname, indexdef 
FROM pg_indexes 
WHERE indexname = 'idx_active_users_email';

-- 或查看所有部分索引
SELECT indexname, indexdef 
FROM pg_indexes 
WHERE indexdef LIKE '%WHERE%';
```

## 删除部分索引

```sql
DROP INDEX IF EXISTS idx_active_users_email;
```

部分索引是 PostgreSQL 的高级特性，合理使用可以显著提高数据库性能，特别是在处理大型表中特定数据子集时。