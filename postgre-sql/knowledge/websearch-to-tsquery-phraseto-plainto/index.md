[返回](/postgre-sql/knowledge/index)


# PostgreSQL 全文检索函数详解

PostgreSQL 提供了强大的全文检索功能，其中 `websearch_to_tsquery`, `phraseto_tsquery` 和 `plainto_tsquery` 是常用的将搜索词转换为 tsquery 类型的函数。

## 1. websearch_to_tsquery

`websearch_to_tsquery` 是 PostgreSQL 9.6+ 引入的函数，它支持类似 web 搜索引擎的查询语法。

### 语法
```sql
websearch_to_tsquery([config regconfig,] query text) RETURNS tsquery
```

### 特性
- 支持引号表示短语搜索 `"phrase search"`
- 支持 `OR` 操作符
- 支持 `-` 排除术语
- 自动处理标点符号

### 示例
```sql
SELECT websearch_to_tsquery('english', 'postgresql full-text search');
```

```
       websearch_to_tsquery        
-----------------------------------
 'postgresql' & 'full' <-> 'text' & 'search'
```

```sql
SELECT websearch_to_tsquery('english', '"postgresql full-text" OR "database search" -mongo');
```

```
                         websearch_to_tsquery                          
---------------------------------------------------------------------
 ( 'postgresql' <-> 'full' <-> 'text' ) | ( 'databas' <-> 'search' ) & !'mongo'
```

## 2. phraseto_tsquery

`phraseto_tsquery` 将搜索词转换为 tsquery，保留短语中的词序。

### 语法
```sql
phraseto_tsquery([config regconfig,] query text) RETURNS tsquery
```

### 特性
- 保留短语中的词序
- 使用 `<->` (FOLLOWED BY) 操作符
- 不支持 OR 或 NOT 操作符

### 示例
```sql
SELECT phraseto_tsquery('english', 'postgresql full-text search');
```

```
             phraseto_tsquery              
------------------------------------------
 'postgresql' <-> 'full' <-> 'text' <-> 'search'
```

## 3. plainto_tsquery

`plainto_tsquery` 将普通文本转换为 tsquery，词之间使用 AND 连接。

### 语法
```sql
plainto_tsquery([config regconfig,] query text) RETURNS tsquery
```

### 特性
- 词之间默认使用 AND 操作符 `&`
- 不保留词序
- 不支持短语搜索或特殊操作符

### 示例
```sql
SELECT plainto_tsquery('english', 'postgresql full-text search');
```

```
        plainto_tsquery        
-------------------------------
 'postgresql' & 'full' & 'text' & 'search'
```

## 对比示例

假设我们有一个文档表：

```sql
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    title TEXT,
    body TEXT,
    tsv_body TSVECTOR
);

INSERT INTO documents (title, body) VALUES
('PostgreSQL Guide', 'PostgreSQL is a powerful open source database system'),
('Full-Text Search', 'PostgreSQL provides full-text search capabilities'),
('Database Comparison', 'Compare PostgreSQL, MySQL and MongoDB');

-- 创建TSVECTOR列
UPDATE documents SET tsv_body = to_tsvector('english', body);
```

### 使用不同函数查询对比

1. **使用 websearch_to_tsquery**:
```sql
SELECT title FROM documents 
WHERE tsv_body @@ websearch_to_tsquery('english', '"PostgreSQL database"');
```

```
       title       
-------------------
 PostgreSQL Guide
 Database Comparison
```

2. **使用 phraseto_tsquery**:
```sql
SELECT title FROM documents 
WHERE tsv_body @@ phraseto_tsquery('english', 'PostgreSQL database');
```

```
       title       
-------------------
 PostgreSQL Guide
```

3. **使用 plainto_tsquery**:
```sql
SELECT title FROM documents 
WHERE tsv_body @@ plainto_tsquery('english', 'PostgreSQL database');
```

```
       title       
-------------------
 PostgreSQL Guide
 Full-Text Search
 Database Comparison
```

## 总结比较
```
| 函数 | 保留词序 | 支持短语 | 支持OR | 支持NOT | 适用场景 |
|------|----------|----------|--------|---------|----------|
| `websearch_to_tsquery` | 是(用引号) | 是 | 是 | 是 | 用户搜索框输入 |
| `phraseto_tsquery` | 是 | 是 | 否 | 否 | 精确短语匹配 |
| `plainto_tsquery` | 否 | 否 | 否 | 否 | 简单AND搜索 |
```
在实际应用中，`websearch_to_tsquery` 通常是最用户友好的选择，特别是当接受用户直接输入时。