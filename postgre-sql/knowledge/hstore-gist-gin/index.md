[返回](/postgre-sql/knowledge/index)

# PostgreSQL hstore 使用 GIST 索引 或 GIN索引

PostgreSQL 中的 `hstore` 是一种键值对数据类型，可以使用 GIST 索引来加速特定类型的查询操作。以下是关于 hstore 使用 GIST 索引的详细介绍：

## hstore 的 GIST 操作符类

PostgreSQL 为 hstore 提供了专门的 GIST 操作符类 `gist_hstore_ops`，支持以下查询操作：

### 支持的运算符
```
| 运算符 | 描述 | 示例 |
|--------|------|------|
| `@>`   | 包含指定的键值对 | `hstore_col @> 'key=>value'` |
| `<@`   | 被指定的键值对包含 | `hstore_col <@ 'key=>value'` |
| `?`    | 包含指定的键 | `hstore_col ? 'key'` |
| `?&`   | 包含所有指定的键 | `hstore_col ?& ARRAY['key1','key2']` |
| `?|`   | 包含任意指定的键 | `hstore_col ?| ARRAY['key1','key2']` |
| `&&`   | 键与数组有重叠 | `hstore_col && ARRAY['key1','key2']` |
```
## 创建 hstore GIST 索引

```sql
-- 启用 hstore 扩展（如果尚未启用）
CREATE EXTENSION IF NOT EXISTS hstore;

-- 创建包含 hstore 列的表
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    attributes HSTORE
);

-- 插入示例数据
INSERT INTO products (name, attributes) VALUES
('Laptop', 'color=>"silver", weight=>"2.5", ram=>"16GB", storage=>"512GB"'),
('Phone', 'color=>"black", weight=>"0.2", ram=>"8GB", storage=>"128GB"'),
('Tablet', 'color=>"white", weight=>"0.5", ram=>"4GB", storage=>"64GB"');

-- 创建 GIST 索引
CREATE INDEX idx_products_attributes ON products USING GIST(attributes gist_hstore_ops);
```

## 使用 hstore GIST 索引的查询示例

### 1. 查询包含特定键值对的记录

```sql
EXPLAIN ANALYZE 
SELECT name, attributes 
FROM products 
WHERE attributes @> 'color=>"black"';
```

可能的执行计划：

```
Index Scan using idx_products_attributes on products  (cost=0.14..8.16 rows=1 width=68) (actual time=0.025..0.026 rows=1 loops=1)
  Index Cond: (attributes @> '"color"=>"black"'::hstore)
Planning Time: 0.098 ms
Execution Time: 0.043 ms
```

### 2. 查询包含任意指定键的记录

```sql
EXPLAIN ANALYZE
SELECT name, attributes
FROM products
WHERE attributes ?| ARRAY['weight', 'battery'];
```

可能的执行计划：

```
Index Scan using idx_products_attributes on products  (cost=0.14..12.15 rows=3 width=68) (actual time=0.021..0.023 rows=3 loops=1)
  Index Cond: (attributes ?| '{weight,battery}'::text[])
Planning Time: 0.075 ms
Execution Time: 0.040 ms
```

### 3. 查询包含所有指定键的记录

```sql
EXPLAIN ANALYZE
SELECT name, attributes
FROM products
WHERE attributes ?& ARRAY['color', 'ram'];
```

可能的执行计划：

```
Index Scan using idx_products_attributes on products  (cost=0.14..12.15 rows=3 width=68) (actual time=0.018..0.020 rows=3 loops=1)
  Index Cond: (attributes ?& '{color,ram}'::text[])
Planning Time: 0.065 ms
Execution Time: 0.035 ms
```

## hstore GIST 索引 vs GiN 索引

对于 hstore 数据类型，PostgreSQL 也支持 GiN 索引，以下是两者的比较：
```
| 特性        | GIST 索引                     | GiN 索引                     |
|------------|-------------------------------|-------------------------------|
| 查询类型   | 支持包含性查询(?/?&/?|/@>/<@) | 支持所有 hstore 操作符       |
| 索引大小   | 通常较小                      | 通常较大                      |
| 构建速度   | 较快                          | 较慢                          |
| 更新性能   | 较好                          | 较差                          |
| 精确度     | 可能返回假阳性                | 精确匹配                      |
```
## 实际查询示例与结果

```sql
-- 查询所有颜色为黑色的产品
SELECT name, attributes->'color' AS color
FROM products
WHERE attributes @> 'color=>"black"';
```

可能的返回结果：

```
 name  | color 
-------+-------
 Phone | black
```

```sql
-- 查询包含 weight 或 battery 键的产品
SELECT name, attributes->'weight' AS weight
FROM products
WHERE attributes ?| ARRAY['weight', 'battery'];
```

可能的返回结果：

```
  name  | weight 
--------+--------
 Laptop | 2.5
 Phone  | 0.2
 Tablet | 0.5
```

## 使用建议

1. **选择 GIST 索引** 当：
   - 主要执行包含性查询(@>, <@, ?, ?&, ?|)
   - 数据更新频繁
   - 索引大小是重要考虑因素

2. **选择 GiN 索引** 当：
   - 需要支持所有 hstore 操作符
   - 数据相对静态
   - 查询性能是首要考虑因素

3. 对于大型 hstore 数据集，可以先测试两种索引类型的性能，再决定使用哪种。