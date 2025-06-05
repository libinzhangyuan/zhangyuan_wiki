[返回](/postgre-sql/knowledge/index)


# PostgreSQL 中的 JSON GIN 索引和 GIN 路径索引

PostgreSQL 为 JSONB 数据类型提供了两种特殊的索引类型：GIN 索引和 GIN 路径索引，它们可以显著提高 JSON 数据的查询性能。

## 1. GIN 索引 (通用倒排索引)

GIN (Generalized Inverted Index) 是 PostgreSQL 为复杂数据类型设计的索引类型，特别适合 JSONB 数据。

### 1.1 创建 GIN 索引

```sql
-- 为整个 JSONB 列创建 GIN 索引
CREATE INDEX idx_gin_attributes ON products USING gin(attributes);
```

### 1.2 GIN 索引支持的查询操作

GIN 索引支持以下 JSONB 操作符：
- `?` - 键是否存在
- `?|` - 任意键存在
- `?&` - 所有键存在
- `@>` - 包含
- `<@` - 被包含
- `=` - 相等

```sql
-- 使用索引的查询示例
EXPLAIN ANALYZE SELECT name FROM products WHERE attributes ? 'brand';
```

```
                                                       QUERY PLAN                                                        
-------------------------------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on products  (cost=12.03..24.06 rows=6 width=32) (actual time=0.025..0.026 rows=3 loops=1)
   Recheck Cond: (attributes ? 'brand'::text)
   Heap Blocks: exact=1
   ->  Bitmap Index Scan on idx_gin_attributes  (cost=0.00..12.03 rows=6 width=0) (actual time=0.018..0.018 rows=3 loops=1)
         Index Cond: (attributes ? 'brand'::text)
 Planning Time: 0.077 ms
 Execution Time: 0.047 ms
```

### 1.3 带 jsonb_path_ops 的 GIN 索引

`jsonb_path_ops` 是 GIN 索引的一种变体，占用空间更小，但只支持 `@>` 操作符。

```sql
CREATE INDEX idx_gin_path_ops ON products USING gin(attributes jsonb_path_ops);
```

## 2. GIN 路径索引

GIN 路径索引是专门为 JSON 路径查询优化的索引类型。

### 2.1 创建 GIN 路径索引

```sql
-- 为特定路径创建 GIN 索引
CREATE INDEX idx_gin_specs_color ON products USING gin((attributes->'specs'->>'color'));
```

### 2.2 使用路径索引的查询

```sql
EXPLAIN ANALYZE SELECT name FROM products WHERE attributes->'specs'->>'color' = '黑色';
```

```
                                                      QUERY PLAN                                                      
----------------------------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on products  (cost=8.27..19.14 rows=8 width=32) (actual time=0.024..0.025 rows=1 loops=1)
   Recheck Cond: ((attributes -> 'specs' ->> 'color'::text) = '黑色'::text)
   Heap Blocks: exact=1
   ->  Bitmap Index Scan on idx_gin_specs_color  (cost=0.00..8.27 rows=8 width=0) (actual time=0.018..0.018 rows=1 loops=1)
         Index Cond: ((attributes -> 'specs' ->> 'color'::text) = '黑色'::text)
 Planning Time: 0.108 ms
 Execution Time: 0.046 ms
```

## 3. 索引选择建议
```
| 索引类型 | 适用场景 | 支持的操作符 | 存储空间 |
|---------|---------|------------|---------|
| 普通 GIN 索引 | 需要查询键是否存在或包含关系 | `?`, `?|`, `?&`, `@>`, `<@`, `=` | 较大 |
| GIN 索引 (jsonb_path_ops) | 主要使用 `@>` 操作符查询 | `@>` | 较小 |
| GIN 路径索引 | 频繁查询特定路径的值 | 路径表达式 | 中等 |
```
## 4. 复合索引示例

可以创建包含多个路径的复合索引：

```sql
CREATE INDEX idx_gin_multi_path ON products USING gin(
    (attributes->'brand'),
    (attributes->'specs'->>'storage')
);
```

## 5. 实际应用示例

### 5.1 电商产品筛选

```sql
-- 创建索引
CREATE INDEX idx_gin_product_filter ON products USING gin(
    (attributes->'brand'),
    (attributes->'specs'),
    (attributes->'tags')
);

-- 使用索引的复杂查询
EXPLAIN ANALYZE SELECT name, price FROM products 
WHERE attributes @> '{"brand": "Apple", "specs": {"storage": "256GB"}}';
```

### 5.2 日志数据分析

```sql
-- 日志表结构
CREATE TABLE server_logs (
    id SERIAL PRIMARY KEY,
    log_time TIMESTAMP,
    log_data JSONB
);

-- 创建路径索引
CREATE INDEX idx_gin_log_status ON server_logs USING gin((log_data->>'status'));
CREATE INDEX idx_gin_log_path ON server_logs USING gin((log_data->>'path'));

-- 查询特定状态码的请求
EXPLAIN ANALYZE SELECT count(*) FROM server_logs 
WHERE log_data->>'status' = '404';
```

## 6. 索引维护提示

1. GIN 索引的更新代价较高，适合读多写少的场景
2. 定期使用 `ANALYZE` 命令更新统计信息
3. 对于大型 JSON 文档，考虑只索引常用路径而非整个文档
4. 可以使用部分索引减少索引大小：

```sql
CREATE INDEX idx_gin_active_products ON products USING gin(attributes)
WHERE price > 5000;
```

通过合理使用 GIN 索引和 GIN 路径索引，可以显著提高 PostgreSQL 中 JSONB 数据的查询性能。