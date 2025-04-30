[返回](/postgre-sql/knowledge/index)


# PostgreSQL 中的 TSVECTOR 类型介绍

TSVECTOR 是 PostgreSQL 中用于全文搜索的特殊数据类型，它表示一个经过词法分析和标准化处理的文档，是 PostgreSQL 全文搜索功能的核心组成部分。

## 基本概念

TSVECTOR 存储了一个排序后的词位(lexemes)列表，每个词位都带有位置信息和权重标记。它主要用于加速文本搜索操作。

## 创建 TSVECTOR

### 1. 使用 `to_tsvector` 函数创建

```sql
SELECT to_tsvector('english', 'The quick brown fox jumped over the lazy dog');
```

返回：
```
                      to_tsvector                      
------------------------------------------------------
 'brown':3 'dog':9 'fox':4 'jump':5 'lazi':8 'quick':2
```

### 2. 从表列创建

假设有一个包含文章的表：

```sql
CREATE TABLE articles (
    id serial primary key,
    title text,
    body text
);

INSERT INTO articles (title, body) VALUES 
('PostgreSQL Tutorial', 'PostgreSQL is a powerful open source database'),
('Advanced PostgreSQL', 'This tutorial covers advanced PostgreSQL features');
```

创建 TSVECTOR 列：

```sql
SELECT id, title, to_tsvector('english', body) AS body_vector 
FROM articles;
```

返回：
```
 id |         title         |                         body_vector                         
----+-----------------------+------------------------------------------------------------
  1 | PostgreSQL Tutorial   | 'databas':7 'open':5 'power':4 'postgresql':1,6 'sourc':6
  2 | Advanced PostgreSQL   | 'advanc':2 'cover':3 'featur':6 'postgresql':4 'tutori':1
```

## TSVECTOR 的组成

TSVECTOR 的格式是：`'词位':位置:权重 '词位':位置 ...`

例如：
```
'fox':4 'jump':5A,7B 'lazi':8
```
- `fox` 出现在位置4
- `jump` 出现在位置5(权重A)和位置7(权重B)
- `lazi` 出现在位置8

## 常用操作

### 1. 连接 TSVECTOR

```sql
SELECT to_tsvector('english', 'hello world') || to_tsvector('english', 'postgresql is great');
```

返回：
```
               ?column?                
---------------------------------------
 'great':6 'hello':1 'postgresql':3 'world':2
```

### 2. 设置权重

```sql
SELECT setweight(to_tsvector('english', 'hello world'), 'A') || 
       setweight(to_tsvector('english', 'postgresql is great'), 'B');
```

返回：
```
                   ?column?                    
----------------------------------------------
 'great':6B 'hello':1A 'postgresql':3B 'world':2A
```

### 3. 获取词位

```sql
SELECT tsvector_to_array(to_tsvector('english', 'The quick brown fox'));
```

返回：
```
    tsvector_to_array    
-------------------------
 {brown,fox,quick}
```

## 实际应用示例

### 1. 创建带有 TSVECTOR 列的表

```sql
CREATE TABLE documents (
    id serial primary key,
    title text,
    content text,
    search_vector tsvector
);

-- 插入数据并自动生成 TSVECTOR
INSERT INTO documents (title, content, search_vector)
VALUES (
    'PostgreSQL Guide', 
    'This is a comprehensive guide to PostgreSQL database',
    to_tsvector('english', 'PostgreSQL Guide This is a comprehensive guide to PostgreSQL database')
);

-- 或者使用生成列
CREATE TABLE documents_auto (
    id serial primary key,
    title text,
    content text,
    search_vector tsvector GENERATED ALWAYS AS (to_tsvector('english', title || ' ' || content)) STORED
);
```

### 2. 使用 GIN 索引加速搜索

```sql
CREATE INDEX idx_documents_search ON documents USING gin(search_vector);
```

### 3. 全文搜索查询

```sql
SELECT title 
FROM documents 
WHERE search_vector @@ to_tsquery('english', 'guide & database');
```

返回：
```
      title      
-----------------
 PostgreSQL Guide
```

## 注意事项

1. TSVECTOR 会自动去除停用词(如 "the", "is", "a" 等)
2. 词位会被标准化处理(如 "jumped" → "jump", "lazy" → "lazi")
3. 不同语言的 TSVECTOR 处理结果不同
4. 对于大型文档，TSVECTOR 可能会占用较多空间

通过合理使用 TSVECTOR 和相应的索引，可以高效地实现复杂的全文搜索功能。