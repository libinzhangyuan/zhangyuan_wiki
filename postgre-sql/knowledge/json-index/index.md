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





# PostgreSQL 中的 JSON B-tree 索引

B-tree 索引是 PostgreSQL 中最常用的索引类型，也可以用于加速 JSON 数据的查询，特别是当查询针对 JSON 中的特定标量值（如字符串、数字）时。

## JSON B-tree 索引的特点

1. **高效**：对于等值查询和范围查询非常高效
2. **适用场景**：适合查询 JSON 中的特定标量值字段
3. **限制**：只能索引单个标量值，不能索引整个 JSON 文档或数组

## 创建 JSON B-tree 索引的基本语法

```sql
CREATE INDEX index_name ON table_name ((json_column->>'key'));
-- 或者对于 jsonb
CREATE INDEX index_name ON table_name ((jsonb_column->>'key'));
```

## 实际示例

### 示例表结构

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    info JSONB
);

INSERT INTO employees (info) VALUES
('{"name": "张三", "age": 28, "department": "研发", "address": {"city": "北京", "street": "中关村"}}'),
('{"name": "李四", "age": 35, "department": "销售", "address": {"city": "上海", "street": "南京路"}}'),
('{"name": "王五", "age": 42, "department": "管理", "address": {"city": "广州", "street": "天河"}}'),
('{"name": "赵六", "age": 31, "department": "研发", "address": {"city": "深圳", "street": "南山"}}');
```

### 1. 对顶级 JSON 字段创建 B-tree 索引

```sql
-- 为部门字段创建索引
CREATE INDEX idx_employee_dept ON employees ((info->>'department'));

-- 为年龄字段创建索引（注意需要转换为整数）
CREATE INDEX idx_employee_age ON employees (((info->>'age')::integer));
```

### 2. 对嵌套 JSON 字段创建 B-tree 索引

```sql
-- 为城市字段创建索引
CREATE INDEX idx_employee_city ON employees ((info#>>'{address,city}'));
```

## 使用 B-tree 索引的查询示例

### 示例 1: 查询特定部门的员工

```sql
EXPLAIN ANALYZE
SELECT info->>'name' AS name, info->>'department' AS department
FROM employees
WHERE info->>'department' = '研发';
```

返回结果：
```
| name | department |
|------|------------|
| 张三 | 研发       |
| 赵六 | 研发       |
```

执行计划会显示使用了我们创建的索引：
```
QUERY PLAN
Index Scan using idx_employee_dept on employees  (cost=0.14..8.16 rows=1 width=64)
  Index Cond: ((info ->> 'department'::text) = '研发'::text)
```

### 示例 2: 查询特定年龄范围的员工

```sql
SELECT info->>'name' AS name, (info->>'age')::int AS age
FROM employees
WHERE (info->>'age')::int BETWEEN 30 AND 40;
```

返回结果：
```
| name | age |
|------|-----|
| 李四 | 35  |
| 赵六 | 31  |
```

### 示例 3: 查询特定城市的员工

```sql
SELECT info->>'name' AS name, info#>>'{address,city}' AS city
FROM employees
WHERE info#>>'{address,city}' = '上海';
```

返回结果：
```
| name | city |
|------|------|
| 李四 | 上海 |
```

## B-tree 索引与 GIN 索引的比较
```
| 特性                | B-tree 索引                     | GIN 索引                     |
|---------------------|--------------------------------|-----------------------------|
| 适用数据类型         | 标量值（字符串、数字等）        | 整个 JSON 文档               |
| 查询类型             | 等值、范围查询                  | 包含、存在性查询             |
| 索引大小             | 较小                           | 较大                        |
| 写入性能             | 较好                           | 较差                        |
| 多值查询             | 不支持                         | 支持                        |
```
## 最佳实践建议

1. **选择合适字段**：只为频繁查询的 JSON 字段创建 B-tree 索引
2. **类型转换**：对于数值字段，确保在索引和查询中使用相同的类型转换
3. **避免过度索引**：只为真正能提升性能的查询创建索引
4. **复合索引**：可以考虑为多个 JSON 字段创建复合 B-tree 索引

```sql
-- 复合索引示例
CREATE INDEX idx_employee_dept_age ON employees (
    (info->>'department'), 
    ((info->>'age')::integer)
);
```

JSON B-tree 索引是优化特定 JSON 字段查询的有效工具，特别是在查询条件明确且针对标量值时，它能提供非常高效的查询性能。