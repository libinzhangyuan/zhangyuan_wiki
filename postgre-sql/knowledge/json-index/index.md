[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的 JSON 索引

JSON 索引是 PostgreSQL 中用于加速 JSON 数据类型查询的特殊索引类型。PostgreSQL 提供了多种 JSON 数据类型（`json` 和 `jsonb`）以及针对这些类型的索引支持。

## JSON 数据类型

PostgreSQL 有两种 JSON 数据类型：

1. `json` - 存储原始的 JSON 数据，保留空格和键顺序
2. `jsonb` (推荐) - 以二进制格式存储解析后的 JSON，支持索引，查询性能更好

## 创建 JSON 索引

### 基本语法

```sql
CREATE INDEX index_name ON table_name USING gin (jsonb_column);
```

### 示例

首先创建一个包含 JSON 数据的表：

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    attributes JSONB
);

INSERT INTO products (name, attributes) VALUES
    ('Laptop', '{"brand": "Apple", "model": "MacBook Pro", "specs": {"cpu": "M1", "ram": 16, "storage": 512}}'),
    ('Phone', '{"brand": "Samsung", "model": "Galaxy S21", "specs": {"cpu": "Exynos 2100", "ram": 8, "storage": 256}}'),
    ('Tablet', '{"brand": "Apple", "model": "iPad Air", "specs": {"cpu": "A14", "ram": 4, "storage": 64}}');
```

### 1. 对整个 JSONB 列创建 GIN 索引

```sql
CREATE INDEX idx_products_attributes ON products USING gin (attributes);
```

这种索引支持所有 JSONB 操作符的查询。

### 2. 对特定 JSON 路径创建索引

```sql
CREATE INDEX idx_products_brand ON products USING btree ((attributes->>'brand'));
```

### 3. 对嵌套 JSON 字段创建索引

```sql
CREATE INDEX idx_products_cpu ON products USING btree ((attributes#>>'{specs,cpu}'));
```

## 使用 JSON 索引的查询示例

### 示例 1: 查询特定品牌的产品

```sql
SELECT name, attributes->>'brand' AS brand 
FROM products 
WHERE attributes->>'brand' = 'Apple';
```

返回结果：
```
| name   | brand |
|--------|-------|
| Laptop | Apple |
| Tablet | Apple |
```

### 示例 2: 查询 CPU 为 M1 的产品

```sql
SELECT name, attributes#>>'{specs,cpu}' AS cpu
FROM products
WHERE attributes#>>'{specs,cpu}' = 'M1';
```

返回结果：
```
| name   | cpu |
|--------|-----|
| Laptop | M1  |
```

### 示例 3: 使用 JSONB 包含操作符 @>

```sql
SELECT name 
FROM products 
WHERE attributes @> '{"brand": "Apple"}';
```

返回结果：
```
| name   |
|--------|
| Laptop |
| Tablet |
```

## JSON 索引类型比较

PostgreSQL 提供了几种 JSON 索引选项：
```
| 索引类型 | 适用场景 | 示例 |
|----------|----------|------|
| GIN 索引 | 通用 JSONB 查询，支持所有 JSONB 操作符 | `CREATE INDEX idx_gin ON table USING gin (jsonb_col);` |
| B-tree 索引 | 特定标量值的等值或范围查询 | `CREATE INDEX idx_btree ON table ((jsonb_col->>'key'));` |
| GIN 路径索引 | 优化特定路径的查询 | `CREATE INDEX idx_path ON table USING gin (jsonb_col jsonb_path_ops);` |
```
## 注意事项

1. JSONB 索引会显著增加存储空间
2. 索引维护会增加写操作的开销
3. 只为频繁查询的路径创建索引
4. 对于简单标量值查询，B-tree 索引通常比 GIN 索引更高效

通过合理使用 JSON 索引，可以显著提高 PostgreSQL 中 JSON 数据的查询性能。




