[返回](/postgre-sql/knowledge/ts-rank/index)

# PostgreSQL 全文搜索排名值预计算方案

预计算和存储排名值可以显著提高全文搜索的性能，特别是在大型文档集或高并发查询场景下。以下是几种实用的预计算方案：

## 方案一：基础预计算存储

### 1. 添加排名值存储列

```sql
ALTER TABLE documents ADD COLUMN search_rank float4;
```

### 2. 预计算并存储排名值

```sql
-- 对现有数据初始化
UPDATE documents 
SET search_rank = ts_rank(tsv, to_tsquery('common_query_terms_here'));

-- 或者使用通用查询
UPDATE documents 
SET search_rank = ts_rank(tsv, to_tsquery('english', 'a | the | and | or'));
```
[通用查询](general-search/index)

### 3. 使用触发器自动维护

```sql
CREATE OR REPLACE FUNCTION update_search_rank()
RETURNS TRIGGER AS $$
BEGIN
    NEW.search_rank := ts_rank(NEW.tsv, to_tsquery('english', 'a | the | and | or'));
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trig_update_search_rank
BEFORE INSERT OR UPDATE OF tsv ON documents
FOR EACH ROW EXECUTE FUNCTION update_search_rank();
```

## 方案二：热门查询预计算

### 1. 创建热门查询排名表

```sql
CREATE TABLE document_query_ranks (
    document_id int REFERENCES documents(id),
    query_text text,
    rank_value float4,
    PRIMARY KEY (document_id, query_text)
);

CREATE INDEX idx_query_ranks ON document_query_ranks(query_text, rank_value DESC);
```

### 2. 定期更新热门查询排名

```sql
-- 假设我们从查询日志中获取了热门查询
WITH popular_queries AS (
    SELECT 'database & design' AS query_text UNION
    SELECT 'postgresql & performance' UNION
    SELECT 'full text search'
)
INSERT INTO document_query_ranks (document_id, query_text, rank_value)
SELECT 
    d.id, 
    pq.query_text, 
    ts_rank(d.tsv, to_tsquery('english', pq.query_text))
FROM 
    documents d
CROSS JOIN 
    popular_queries pq
ON CONFLICT (document_id, query_text) 
DO UPDATE SET rank_value = EXCLUDED.rank_value;
```

## 方案三：分层预计算

### 1. 创建摘要表存储常用排名

```sql
CREATE MATERIALIZED VIEW document_search_summary AS
SELECT
    id,
    title,
    ts_rank(tsv, to_tsquery('english', 'a | the | and | or')) AS base_rank,
    ts_rank_cd(tsv, to_tsquery('english', 'a | the | and | or')) AS cover_rank,
    last_updated TIMESTAMP
FROM documents;

-- 创建索引加速查询
CREATE INDEX idx_summary_base_rank ON document_search_summary(base_rank);
CREATE INDEX idx_summary_cover_rank ON document_search_summary(cover_rank);
```

### 2. 定期刷新物化视图

```sql
-- 手动刷新
REFRESH MATERIALIZED VIEW CONCURRENTLY document_search_summary;

-- 或者设置定时任务(使用pg_cron扩展)
SELECT cron.schedule('0 3 * * *', $$REFRESH MATERIALIZED VIEW CONCURRENTLY document_search_summary$$);
```

## 方案四：动态+静态混合方案

### 1. 存储部分计算结果

```sql
ALTER TABLE documents ADD COLUMN lexeme_stats jsonb;

-- 更新统计信息
UPDATE documents SET lexeme_stats = jsonb_build_object(
    'length', length(tsv::text),
    'unique_lexemes', array_length(tsvector_to_array(tsv), 1),
    'total_lexemes', SELECT COUNT(*) FROM unnest(tsvector_to_array(tsv))
);
```

### 2. 查询时快速计算

```sql
SELECT 
    id, 
    title,
    -- 使用预存数据加速计算
    ts_rank(tsv, query, 1) * (1.0 / (lexeme_stats->>'length')::float) AS norm_rank
FROM 
    documents, 
    to_tsquery('search & term') query
WHERE 
    tsv @@ query
ORDER BY 
    norm_rank DESC;
```

## 方案五：分区预计算

### 1. 按文档类别分区预计算

```sql
CREATE TABLE document_ranks_by_category (
    category_id int,
    document_id int,
    base_rank float4,
    PRIMARY KEY (category_id, document_id)
) PARTITION BY LIST(category_id);

-- 为每个类别创建分区
CREATE TABLE doc_ranks_tech PARTITION OF document_ranks_by_category
    FOR VALUES IN (1);
CREATE TABLE doc_ranks_science PARTITION OF document_ranks_by_category
    FOR VALUES IN (2);
-- 更多分区...

-- 初始化数据
INSERT INTO document_ranks_by_category
SELECT 
    category_id, 
    id, 
    ts_rank(tsv, to_tsquery('english', 'a | the | and | or'))
FROM 
    documents;
```

## 维护策略

1. **定时批量更新**:
   ```sql
   -- 每天凌晨更新
   UPDATE documents 
   SET search_rank = ts_rank(tsv, to_tsquery('english', 'a | the | and | or'))
   WHERE last_modified > CURRENT_DATE - INTERVAL '1 day';
   ```

2. **增量更新**:
   ```sql
   -- 只更新变化的内容
   WITH changed_docs AS (
       SELECT id FROM documents 
       WHERE tsv IS DISTINCT FROM last_indexed_tsv
       LIMIT 1000
   )
   UPDATE documents d
   SET 
       search_rank = ts_rank(d.tsv, to_tsquery('english', 'a | the | and | or')),
       last_indexed_tsv = d.tsv
   FROM changed_docs cd
   WHERE d.id = cd.id;
   ```

3. **读写分离**:
   - 在主库上计算排名值
   - 将结果同步到只读副本
   - 查询操作使用只读副本

## 性能优化建议

1. **索引优化**:
   ```sql
   CREATE INDEX idx_combined_search ON documents 
   USING gin(tsv) INCLUDE (search_rank, title);
   ```

2. **部分索引**:
   ```sql
   CREATE INDEX idx_high_rank_docs ON documents (id)
   WHERE search_rank > 0.5;
   ```

3. **查询优化**:
   ```sql
   EXPLAIN ANALYZE 
   SELECT id, title FROM documents
   WHERE tsv @@ to_tsquery('search & term')
   ORDER BY search_rank DESC
   LIMIT 20;
   ```

预计算方案的选择应基于您的具体需求、数据量和查询模式。对于大多数应用，方案一或方案三提供了良好的平衡点，而高流量站点可能需要更复杂的方案二或方案四。