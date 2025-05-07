[返回](/postgre-sql/knowledge/index)

# PostgreSQL 全文搜索类型详解

PostgreSQL 提供了强大的全文搜索功能，主要通过 `tsvector` 和 `tsquery` 两种数据类型实现。以下是详细的介绍和示例：

## 1. 全文搜索相关数据类型

### `tsvector` 类型
- 表示一个被优化的文本搜索向量
- 会自动去除停用词（如"的"、"是"等）并对词干进行标准化

**示例：**
```sql
SELECT 'PostgreSQL是一种强大的开源关系型数据库'::tsvector;
```
**返回结果**：
```
"'postgresql是一种强大的开源关系型数据库'"
```

### `tsquery` 类型
- 表示文本搜索查询条件
- 支持逻辑操作符：& (AND), | (OR), ! (NOT)

**示例：**
```sql
SELECT '开源 & 数据库'::tsquery;
```
**返回结果**：
```
'开源' & '数据库'
```

## 2. 基本全文搜索操作

### 创建支持全文搜索的表
```sql
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title TEXT,
    content TEXT,
    content_search TSVECTOR GENERATED ALWAYS AS (to_tsvector('chinese', content)) STORED
);
```

### 插入数据
```sql
INSERT INTO articles (title, content) VALUES 
('PostgreSQL简介', 'PostgreSQL是一种功能强大的开源对象关系数据库系统');
```

### 基本搜索
```sql
SELECT title FROM articles WHERE content_search @@ to_tsquery('chinese', '开源 & 数据库');
```
**返回结果**：
```
    title
--------------
 PostgreSQL简介
```

## 3. 高级搜索功能

### 权重分配
```sql
SELECT title, 
       ts_rank_cd(
          to_tsvector('chinese', title || ' ' || content),
          to_tsquery('chinese', '数据库')
       ) AS rank
FROM articles
ORDER BY rank DESC;
```

### 高亮显示结果
```sql
SELECT title, 
       ts_headline('chinese', content, to_tsquery('chinese', '数据库'),
           'StartSel=<b>, StopSel=</b>') AS highlighted
FROM articles;
```

## 4. 配置全文搜索

### 查看可用配置
```sql
SELECT cfgname FROM pg_catalog.pg_ts_config;
```

### 中文搜索配置
PostgreSQL默认不包含完善的中文配置，可以使用zhparser扩展：
```sql
CREATE EXTENSION zhparser;
CREATE TEXT SEARCH CONFIGURATION chinese (PARSER = zhparser);
ALTER TEXT SEARCH CONFIGURATION chinese ADD MAPPING FOR n,v,a,i,e,l WITH simple;
```

## 5. 索引优化

为提高全文搜索性能，可以创建GIN索引：
```sql
CREATE INDEX idx_articles_search ON articles USING GIN(content_search);
```

## 6. 实际应用示例

### 案例：新闻搜索系统
```sql
-- 创建表
CREATE TABLE news (
    id SERIAL PRIMARY KEY,
    title TEXT,
    body TEXT,
    pub_date TIMESTAMP,
    search_vector TSVECTOR GENERATED ALWAYS AS (
        setweight(to_tsvector('chinese', coalesce(title, '')), 'A') || 
        setweight(to_tsvector('chinese', coalesce(body, '')), 'B')
    ) STORED
);

-- 创建索引
CREATE INDEX idx_news_search ON news USING GIN(search_vector);

-- 插入数据
INSERT INTO news (title, body, pub_date) VALUES
('PostgreSQL 15发布', 'PostgreSQL全球开发组今天宣布PostgreSQL 15发布...', '2022-10-13');

-- 执行搜索
SELECT title, pub_date, 
       ts_headline('chinese', body, to_tsquery('chinese', '发布'), 
           'StartSel=<mark>, StopSel=</mark>') AS snippet
FROM news
WHERE search_vector @@ to_tsquery('chinese', 'PostgreSQL & 发布')
ORDER BY ts_rank(search_vector, to_tsquery('chinese', 'PostgreSQL & 发布')) DESC;
```

## 7. 注意事项

1. 中文搜索需要额外配置，可以使用zhparser或pg_jieba等扩展
2. 对于大型文档，考虑将内容分解为多个tsvector列以提高性能
3. 定期维护（VACUUM ANALYZE）以保持搜索效率
4. 考虑使用CONCURRENTLY创建索引以避免锁表

PostgreSQL的全文搜索功能强大且灵活，可以满足从简单到复杂的各种文本搜索需求。