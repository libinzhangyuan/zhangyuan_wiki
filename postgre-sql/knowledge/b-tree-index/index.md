[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的 B-tree 索引

B-tree (Balanced Tree，平衡树) 是 PostgreSQL 中最常用和默认的索引类型，适用于大多数标准查询场景。

## B-tree 索引基本特性

1. **平衡结构**：保持数据平衡，确保查询效率稳定
2. **有序存储**：索引条目按键值排序存储
3. **高效查询**：
   - 等值查询：`=`
   - 范围查询：`>`, `<`, `BETWEEN`
   - 前缀查询：`LIKE 'abc%'`
4. **多列支持**：支持复合索引(多列组合)

## 创建 B-tree 索引

### 基本语法

```sql
CREATE INDEX index_name ON table_name (column_name);
```

### 多列索引

```sql
CREATE INDEX idx_name ON table_name (col1, col2, col3);
```

### 指定排序方式

```sql
CREATE INDEX idx_name ON table_name (col1 DESC, col2 ASC);
```

## B-tree 索引适用场景

1. **高选择性列**：列中不同值多、重复值少
   ```sql
   -- 适合创建B-tree索引
   CREATE INDEX idx_users_email ON users(email);
   ```

2. **排序操作**：经常用于 `ORDER BY` 的列
   ```sql
   -- 优化排序查询
   CREATE INDEX idx_orders_date ON orders(order_date);
   ```

3. **范围查询**：
   ```sql
   -- 高效执行范围查询
   SELECT * FROM logs WHERE created_at BETWEEN '2023-01-01' AND '2023-01-31';
   ```

4. **连接操作**：连接条件列
   ```sql
   -- 优化连接性能
   CREATE INDEX idx_orders_user_id ON orders(user_id);
   ```

## B-tree 索引内部结构

```
          [根节点]
         /    |    \
    [内部节点] [内部节点] [内部节点]
     /  |  \          /  |  \
[叶子节点]...      [叶子节点]...
```

1. **叶子节点**：存储键值和指向表行的TID(元组ID)
2. **内部节点**：存储导航信息，用于快速定位叶子节点
3. **节点大小**：通常8KB，与PostgreSQL页面大小一致

## B-tree 索引特殊功能

### 1. 索引覆盖扫描(Index-Only Scan)

当查询只访问索引包含的列时，PostgreSQL可以直接从索引获取数据：

```sql
-- 如果name和id都在索引中
CREATE INDEX idx_users_id_name ON users(id, name);

-- 可能使用index-only scan
SELECT id, name FROM users WHERE id BETWEEN 100 AND 200;
```

### 2. NULL值处理

```sql
-- 包含NULL值的索引
CREATE INDEX idx_products_price ON products(price);

-- 查找NULL值
SELECT * FROM products WHERE price IS NULL;
```

### 3. 唯一索引

```sql
CREATE UNIQUE INDEX idx_users_username ON users(username);
```

## B-tree 索引优化技巧

1. **选择性高的列优先**：将选择性高的列放在复合索引前面
   ```sql
   -- 好: state只有几种值，zip_code变化多
   CREATE INDEX idx_address ON addresses(zip_code, state);
   ```

2. **索引列顺序**：按照查询条件顺序创建复合索引
   ```sql
   -- 对于 WHERE a = ? AND b > ? 查询
   CREATE INDEX idx_tab ON table(a, b);
   ```

3. **部分索引**：只为部分数据创建索引
   ```sql
   -- 只为活跃用户创建索引
   CREATE INDEX idx_active_users ON users(email) WHERE is_active = true;
   ```

4. **函数索引**：对表达式创建索引
   ```sql
   -- 不区分大小写的查询
   CREATE INDEX idx_users_lower_name ON users(lower(name));
   ```

## B-tree 索引限制

1. **不适合**：
   - 频繁更新的列
   - 低选择性列(如布尔值)
   - 大型对象(考虑使用hash或GIN索引)

2. **空间占用**：索引会占用额外存储空间

3. **维护成本**：插入/更新/删除操作需要维护索引

## 管理 B-tree 索引

### 查看索引

```sql
-- 查看表的所有索引
SELECT * FROM pg_indexes WHERE tablename = 'your_table';

-- 查看索引大小
SELECT pg_size_pretty(pg_total_relation_size('index_name')) AS index_size;
```

### 重建索引

```sql
-- 重建索引(解决膨胀问题)
REINDEX INDEX index_name;

-- 重建表的所有索引
REINDEX TABLE table_name;
```

### 删除索引

```sql
DROP INDEX IF EXISTS index_name;
```

B-tree 索引是 PostgreSQL 中最通用的索引类型，合理使用可以显著提高查询性能，但也需要根据实际查询模式和数据特性进行适当设计和维护。