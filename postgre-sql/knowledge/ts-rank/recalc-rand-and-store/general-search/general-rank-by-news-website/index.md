[返回](/postgre-sql/knowledge/ts-rank/recalc-rand-and-store/general-search/index)

# 在SELECT查询中使用新闻网站通用排名

在新闻网站中，通用排名可以用于多种查询场景，下面我将详细介绍如何在SELECT语句中有效利用预计算的通用排名。

## 基础用法

### 1. 简单排序查询
```sql
SELECT 
    id,
    title,
    publish_time,
    generic_rank AS news_rank
FROM 
    news_articles
ORDER BY 
    generic_rank DESC
LIMIT 20;
```

### 2. 结合分类筛选
```sql
SELECT 
    id,
    title,
    category,
    generic_rank
FROM 
    news_articles
WHERE 
    category = '政治'
ORDER BY 
    generic_rank DESC
LIMIT 10;
```

## 高级用法

### 3. 结合实时点击数据
```sql
SELECT 
    n.id,
    n.title,
    -- 组合预计算排名和实时点击数据
    (n.generic_rank * 0.7 + LOG(2 + c.hourly_clicks) * 0.3) AS dynamic_rank
FROM 
    news_articles n
LEFT JOIN 
    article_clicks c ON n.id = c.article_id
ORDER BY 
    dynamic_rank DESC
LIMIT 15;
```

### 4. 分时段加权排名
```sql
SELECT 
    id,
    title,
    -- 早间时段提高时效性权重
    CASE 
        WHEN EXTRACT(HOUR FROM CURRENT_TIME) BETWEEN 6 AND 9 
        THEN generic_rank * 1.2
        ELSE generic_rank
    END AS time_adjusted_rank
FROM 
    news_articles
ORDER BY 
    time_adjusted_rank DESC
LIMIT 10;
```

## 混合排名策略

### 5. 通用排名 + 全文搜索
```sql
SELECT 
    id,
    title,
    -- 组合通用排名和搜索相关性
    0.4 * generic_rank + 
    0.6 * ts_rank(tsv, websearch_to_tsquery('中美关系')) AS combined_rank,
    snippet(tsv, websearch_to_tsquery('中美关系'), '...', 1, 20) AS highlight
FROM 
    news_articles
WHERE 
    tsv @@ websearch_to_tsquery('中美关系')
ORDER BY 
    combined_rank DESC
LIMIT 10;
```

### 6. 个性化推荐
```sql
SELECT 
    n.id,
    n.title,
    -- 结合用户偏好和通用排名
    (n.generic_rank * 0.6 + 
     COALESCE(up.preference_score, 0.5) * 0.4) AS personalized_rank
FROM 
    news_articles n
LEFT JOIN 
    user_preferences up ON n.category = up.category AND up.user_id = 123
ORDER BY 
    personalized_rank DESC
LIMIT 15;
```

## 分页查询优化

### 7. 高效分页
```sql
-- 第一页
SELECT id, title, generic_rank
FROM news_articles
ORDER BY generic_rank DESC
LIMIT 20;

-- 后续页 (使用游标)
SELECT id, title, generic_rank
FROM news_articles
WHERE generic_rank < :last_rank_value
ORDER BY generic_rank DESC
LIMIT 20;
```

## 实时动态调整

### 8. 结合最新评论
```sql
SELECT 
    n.id,
    n.title,
    -- 增加评论活跃度因素
    (n.generic_rank * 0.8 + 
     LOG(2 + COUNT(c.id)) * 0.2) AS social_rank
FROM 
    news_articles n
LEFT JOIN 
    comments c ON n.id = c.article_id AND c.created_at > NOW() - INTERVAL '6 hours'
GROUP BY 
    n.id
ORDER BY 
    social_rank DESC
LIMIT 10;
```

## 注意事项

1. **索引优化**：
   ```sql
   CREATE INDEX idx_news_generic_rank ON news_articles(generic_rank DESC);
   CREATE INDEX idx_news_category_rank ON news_articles(category, generic_rank DESC);
   ```

2. **定期刷新**：
   ```sql
   -- 每天凌晨更新排名
   UPDATE news_articles SET generic_rank = (...计算公式...)
   WHERE publish_time > CURRENT_DATE - INTERVAL '30 days';
   ```

3. **查询性能监控**：
   ```sql
   EXPLAIN ANALYZE 
   SELECT id, title FROM news_articles 
   ORDER BY generic_rank DESC LIMIT 10;
   ```

通过合理使用预计算的通用排名，可以显著提高新闻网站查询性能，同时保持内容的相关性和时效性。实际应用中，建议根据具体业务需求调整排名公式和权重分配。