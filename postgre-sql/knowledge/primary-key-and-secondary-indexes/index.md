[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的主键(Primary Keys)和二级索引(Secondary Indexes)

## 主键(Primary Keys)

### 定义与特性
- **唯一标识符**：主键是表中唯一标识每一行的列或列组合
- **非空约束**：主键列不能包含NULL值
- **自动创建索引**：PostgreSQL会自动为主键创建唯一索引
- **表只能有一个主键**

### 创建主键

```sql
-- 创建表时定义主键
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL
);

-- 为现有表添加主键
ALTER TABLE orders ADD PRIMARY KEY (order_id);

-- 复合主键
CREATE TABLE order_items (
    order_id INT,
    item_id INT,
    quantity INT,
    PRIMARY KEY (order_id, item_id)
);
```

### 主键索引特点
1. 默认创建的是B-tree索引
2. 支持唯一性约束检查
3. 通常用作表的聚集索引(虽然PostgreSQL没有真正的聚集索引概念)

## 二级索引(Secondary Indexes)

### 定义与特性
- **辅助访问路径**：提供除主键外的其他查询路径
- **可选创建**：根据查询需求决定是否创建
- **不影响数据唯一性**：除非显式指定UNIQUE约束
- **一个表可以有多个二级索引**

### 常见二级索引类型

1. **B-tree索引** (默认类型)
   ```sql
   CREATE INDEX idx_users_username ON users(username);
   ```

2. **Hash索引**
   ```sql
   CREATE INDEX idx_users_email_hash ON users USING hash(email);
   ```

3. **GiST索引** (通用搜索树)
   ```sql
   CREATE INDEX idx_properties_geom ON properties USING gist(geom);
   ```

4. **GIN索引** (倒排索引)
   ```sql
   CREATE INDEX idx_docs_content ON docs USING gin(to_tsvector('english', content));
   ```

5. **BRIN索引** (块范围索引)
   ```sql
   CREATE INDEX idx_logs_timestamp ON logs USING brin(timestamp);
   ```

### 二级索引创建示例

```sql
-- 单列索引
CREATE INDEX idx_products_price ON products(price);

-- 多列复合索引
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);

-- 包含列的索引(PostgreSQL 11+)
CREATE INDEX idx_orders_with_total ON orders(order_date) INCLUDE (total_amount);

-- 部分索引(只索引满足条件的行)
CREATE INDEX idx_active_users ON users(username) WHERE is_active = true;

-- 表达式索引
CREATE INDEX idx_users_lower_name ON users(lower(username));
```

## 主键索引与二级索引对比
```
| 特性                | 主键索引                     | 二级索引                     |
|---------------------|----------------------------|----------------------------|
| 唯一性              | 总是唯一                    | 除非显式声明UNIQUE          |
| NULL值              | 不允许                      | 允许(除非特别约束)          |
| 数量限制            | 每表一个                    | 每表多个                    |
| 自动创建            | 定义主键时自动创建           | 需显式创建                  |
| 主要用途            | 行唯一标识/关系建立          | 优化查询性能                |
| 索引类型            | 通常是B-tree                | 可以是多种类型              |
| 是否强制            | 必须                        | 可选                        |
```
## 索引使用最佳实践

1. **选择性高的列优先**：高基数(不同值多)的列更适合建索引
2. **考虑查询模式**：为WHERE、JOIN、ORDER BY中频繁使用的列建索引
3. **复合索引列顺序**：将最常用于查询条件的列放在前面
4. **避免过多索引**：每个索引会增加写操作的开销
5. **监控索引使用**：
   ```sql
   SELECT * FROM pg_stat_user_indexes;
   ```
6. **定期维护**：
   ```sql
   ANALYZE table_name;  -- 更新统计信息
   REINDEX INDEX index_name;  -- 重建索引
   ```

## 索引使用示例

```sql
-- 查看表的所有索引
SELECT indexname, indexdef 
FROM pg_indexes 
WHERE tablename = 'users';

-- 解释查询计划查看索引使用情况
EXPLAIN ANALYZE SELECT * FROM users WHERE username = 'john_doe';

-- 删除索引
DROP INDEX IF EXISTS idx_users_username;
```

正确使用主键和二级索引可以显著提高PostgreSQL数据库的查询性能，但需要根据实际应用场景和数据访问模式进行合理设计。