[返回](/postgre-sql/knowledge/index)

# 通用倒排索引 GIN 索引详解

通用倒排索引 GIN (Generalized Inverted Index) 是 PostgreSQL 中的一种高效索引类型，特别适合用于包含多个组件值的数据类型，如数组、JSONB、全文搜索等。

## 1. GIN 索引基本概念
```
GIN 是"通用倒排索引"的缩写，它的主要特点是：
- 适用于需要索引复合值的数据类型
- 支持"包含"查询（如 `@>`、`<@` 等操作符）
- 对数组、JSONB、范围类型和全文搜索特别有效
```
## 2. 创建 GIN 索引的基本语法

```sql
CREATE INDEX index_name ON table_name USING GIN (column_name);
```

## 3. GIN 索引适用场景及示例

### 3.1 数组类型索引

**示例表：**
```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    tags TEXT[]
);
```

**创建 GIN 索引：**
```sql
CREATE INDEX idx_products_tags ON products USING GIN (tags);
```

**查询示例：**
```sql
-- 查找包含"电子"标签的产品
SELECT * FROM products WHERE tags @> ARRAY['电子'];

-- 查找标签中包含"电子"或"家电"的产品
SELECT * FROM products WHERE tags && ARRAY['电子', '家电'];
```

**返回结果示例：**
```
 id |      name      |       tags       
----+----------------+------------------
  1 | 智能手机       | {电子,通讯,数码}
  3 | 智能电视       | {电子,家电,数码}
```

### 3.2 JSONB 类型索引

**示例表：**
```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    info JSONB
);
```

**创建 GIN 索引：**
```sql
CREATE INDEX idx_orders_info ON orders USING GIN (info);
```

**查询示例：**
```sql
-- 查找info中包含status字段且值为'shipped'的订单
SELECT * FROM orders WHERE info @> '{"status": "shipped"}';

-- 查找info中items数组包含"book"的订单
SELECT * FROM orders WHERE info @> '{"items": ["book"]}';
```

**返回结果示例：**
```
 id |                               info                               
----+------------------------------------------------------------------
  2 | {"status": "shipped", "items": ["book", "pen"], "price": 45.99}
  4 | {"status": "shipped", "items": ["book"], "price": 29.99}
```

### 3.3 全文搜索索引

**示例表：**
```sql
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title TEXT,
    content TEXT,
    tsv TSVECTOR
);
```

**创建 GIN 索引：**
```sql
-- 首先创建TSVECTOR列（或直接索引表达式）
UPDATE articles SET tsv = to_tsvector('english', coalesce(title,'') || ' ' || coalesce(content,''));

CREATE INDEX idx_articles_tsv ON articles USING GIN (tsv);
```

**查询示例：**
```sql
-- 搜索包含"database"和"performance"的文章
SELECT title FROM articles WHERE tsv @@ to_tsquery('english', 'database & performance');
```

**返回结果示例：**
```
               title               
-----------------------------------
 PostgreSQL 性能优化指南
 数据库调优技巧
```

## 4. GIN 索引与 B-tree 索引对比
```
| 特性                | GIN 索引                     | B-tree 索引               |
|---------------------|-----------------------------|--------------------------|
| 适用数据类型        | 数组、JSONB、全文搜索等      | 标量值（数字、字符串等）  |
| 查询类型            | 包含查询、重叠查询等         | 等值查询、范围查询        |
| 索引大小            | 通常较大                     | 通常较小                  |
| 更新性能            | 较慢                         | 较快                      |
| NULL 值处理         | 不索引NULL值                 | 可以索引NULL值            |
```
## 5. GIN 索引优化技巧

1. **使用 GIN 索引的快速更新技术**：
   ```sql
   CREATE INDEX idx_name ON table_name USING GIN (column_name) WITH (fastupdate = on);
   ```

2. **对 JSONB 使用路径索引**：
   ```sql
   CREATE INDEX idx_jsonb_path ON table_name USING GIN ((column_name->'path'->'to'->>'field'));
   ```

3. **部分索引**（只索引满足条件的数据）：
   ```sql
   CREATE INDEX idx_partial ON table_name USING GIN (column_name) WHERE condition;
   ```

4. **多列复合 GIN 索引**：
   ```sql
   CREATE INDEX idx_multi ON table_name USING GIN (column1, column2);
   ```

## 6. GIN 索引的限制

1. 相比 B-tree 索引，GIN 索引占用更多空间
2. 索引更新开销较大，不适合频繁写入的场景
3. 不支持排序操作
4. 对简单等值查询效率不如 B-tree 索引

## 7. 实际应用案例

**案例：电商平台商品搜索**

```sql
-- 创建表
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    attributes JSONB,
    categories TEXT[],
    search_terms TSVECTOR
);

-- 创建多种GIN索引
CREATE INDEX idx_products_attrs ON products USING GIN (attributes);
CREATE INDEX idx_products_cats ON products USING GIN (categories);
CREATE INDEX idx_products_search ON products USING GIN (search_terms);

-- 复杂查询示例
SELECT name, attributes->>'price' AS price
FROM products
WHERE attributes @> '{"brand": "Apple"}'
AND categories && ARRAY['Electronics', 'Mobile']
AND search_terms @@ to_tsquery('english', 'iphone & (pro | max)');
```

**返回结果示例：**
```
        name         | price  
---------------------+--------
 iPhone 13 Pro       | 999.00
 iPhone 13 Pro Max   | 1099.00
```

GIN 索引是 PostgreSQL 中处理复杂数据类型的强大工具，合理使用可以显著提高查询性能，特别是在处理半结构化数据和全文搜索场景下。