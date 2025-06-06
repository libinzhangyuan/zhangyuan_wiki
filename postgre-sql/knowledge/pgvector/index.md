[返回](/postgre-sql/knowledge/index)

# PostgreSQL pgvector 扩展介绍

pgvector 是 PostgreSQL 的一个扩展，用于存储和查询向量数据，特别适合机器学习领域的向量相似性搜索。

## 安装 pgvector

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

## 基本功能

### 1. 创建向量列

```sql
CREATE TABLE items (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    embedding VECTOR(3)  -- 3维向量
);
```

### 2. 插入向量数据

```sql
INSERT INTO items (name, embedding) VALUES 
('item1', '[1,2,3]'),
('item2', '[4,5,6]'),
('item3', '[7,8,9]');
```

### 3. 查询向量数据

```sql
SELECT * FROM items;
```

返回结果：
```
 id | name  | embedding 
----+-------+-----------
  1 | item1 | [1,2,3]
  2 | item2 | [4,5,6]
  3 | item3 | [7,8,9]
```

## 向量操作

### 1. 计算欧式距离 (L2距离)

```sql
SELECT id, name, embedding <-> '[3,1,2]' AS distance 
FROM items 
ORDER BY distance;
```

返回结果：
```
 id | name  |     distance     
----+-------+------------------
  1 | item1 | 2.449489742783178
  2 | item2 | 4.123105625617661
  3 | item3 | 7.810249675906654
```

### 2. 计算内积

```sql
SELECT id, name, embedding <#> '[3,1,2]' AS inner_product 
FROM items 
ORDER BY inner_product;
```

返回结果：
```
 id | name  | inner_product 
----+-------+---------------
  3 | item3 |          47
  2 | item2 |          28
  1 | item1 |          10
```

### 3. 计算余弦相似度

```sql
SELECT id, name, embedding <=> '[3,1,2]' AS cosine_similarity 
FROM items 
ORDER BY cosine_similarity DESC;
```

返回结果：
```
 id | name  | cosine_similarity  
----+-------+--------------------
  1 | item1 | 0.9258200997725514
  2 | item2 | 0.9025767755554449
  3 | item3 | 0.8784584686522417
```

## 近似最近邻搜索 (ANN)

pgvector 支持高效的近似最近邻搜索，特别适合高维向量。

### 1. 创建索引

```sql
CREATE INDEX ON items USING ivfflat (embedding vector_l2_ops) WITH (lists = 100);
```

### 2. 近似搜索

```sql
SELECT id, name, embedding <-> '[3,1,2]' AS distance 
FROM items 
ORDER BY embedding <-> '[3,1,2]'
LIMIT 2;
```

返回结果：
```
 id | name  |     distance     
----+-------+------------------
  1 | item1 | 2.449489742783178
  2 | item2 | 4.123105625617661
```

## 实际应用示例

### 1. 图像相似性搜索

```sql
CREATE TABLE images (
    id SERIAL PRIMARY KEY,
    url TEXT,
    features VECTOR(512)
);

-- 创建索引
CREATE INDEX ON images USING ivfflat (features vector_l2_ops) WITH (lists = 100);

-- 查询相似图像
SELECT id, url, features <=> '[0.1,0.2,...,0.5]' AS similarity
FROM images
ORDER BY similarity DESC
LIMIT 5;
```

### 2. 文本语义搜索

```sql
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    title TEXT,
    content TEXT,
    embedding VECTOR(768)
);

-- 查询相关文档
SELECT id, title, embedding <=> '[0.3,0.1,...,0.8]' AS similarity
FROM documents
ORDER BY similarity DESC
LIMIT 10;
```

## 性能优化技巧

1. 选择合适的索引类型：`ivfflat` 适合大多数场景，`hnsw` 提供更高的召回率但构建时间更长

2. 调整 `lists` 参数：更大的值提高准确性但降低查询速度

3. 对于高维向量，考虑使用 PCA 降维

4. 定期使用 `ANALYZE` 更新统计信息

pgvector 为 PostgreSQL 提供了强大的向量搜索能力，使其成为机器学习应用和相似性搜索场景的理想数据库选择。