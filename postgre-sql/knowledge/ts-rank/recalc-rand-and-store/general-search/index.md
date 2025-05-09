[返回](/postgre-sql/knowledge/ts-rank/recalc-rand-and-store/index)


# PostgreSQL 全文搜索中的"通用查询"解释

在这个预计算排名值的SQL语句中，所谓的"通用查询"是指一个包含常见英语停用词的查询，目的是为所有文档计算一个基础排名值。

## 详细解释

```sql
UPDATE documents 
SET search_rank = ts_rank(tsv, to_tsquery('english', 'a | the | and | or'));
```

### 1. 什么是通用查询？

这里的"通用查询"特指：
- 使用最常见的英语停用词（a, the, and, or）组成的查询
- 这些词几乎出现在所有文档中
- 目的是为所有文档计算一个非零的基础排名值

### 2. 为什么这样设计？

这种设计有几个目的：

1. **预先计算非零排名**：
   - 确保所有文档都有一个基准排名值
   - 避免后续查询时出现零排名的情况

2. **反映文档基本特征**：
   - 长文档通常会包含更多这些常见词
   - 因此会获得比短文档稍高的基础排名

3. **加速后续查询**：
   - 可以作为更具体查询的基数
   - 可以与其他排名因素结合使用

### 3. 工作原理

当执行 `ts_rank(tsv, to_tsquery('a | the | and | or'))` 时：

1. 系统会检查文档的tsvector中是否包含这些词
2. 由于这些词非常常见，几乎所有文档都会匹配
3. 排名计算会考虑：
   - 这些词的出现频率
   - 它们在文档中的分布
   - 文档的总长度

### 4. 实际效果

- 长文档通常会获得比短文档稍高的基础排名
- 内容丰富、词汇多样的文档排名会更高
- 内容很少的文档排名会较低

### 5. 更好的替代方案

更精确的通用排名可以使用：

```sql
-- 使用文档自身的特征计算基础排名
UPDATE documents SET search_rank = (
  SELECT 1.0 / (1.0 + ln(1.0 + length(tsv::text)/10.0)
);
```

或者结合文档元数据：

```sql
UPDATE documents SET search_rank = (
  0.3 * (COALESCE(quality_score, 0.5)) +
  0.7 * (1.0 / (1.0 + ln(1.0 + length(tsv::text)/10.0))
);
```

### 6. 使用场景

这种通用排名适合：
- 需要快速排序但无具体搜索词时
- 作为复合排名的一部分
- 在个性化搜索中作为基础分数

### 7. 注意事项

1. 这种通用排名不能替代具体查询的实时排名计算
2. 对于精确搜索，仍需要结合具体查询词计算
3. 可以考虑定期更新（如每天）而不是每次查询都更新

这种预计算方法最适合内容相对静态、查询模式可预测的应用场景。对于高度动态的内容或查询，可能需要更复杂的策略。


<br><br>


# PostgreSQL 通用排名实例详解

通用排名是为文档预先计算的基础相关性分数，不考虑具体搜索词，而是基于文档本身的特征。以下是几种实用的通用排名计算方法和具体示例：

## 1. 基础文档特征排名

### 示例1：基于文档长度
```sql
-- 文档越短排名越高（适合摘要类内容）
UPDATE documents SET generic_rank = 1.0 / (1.0 + length(body)::float/1000);

-- 文档长度适中排名高（避免过长或过短）
UPDATE documents SET generic_rank = 
  1.0 - abs(length(body)::float - 1000) / GREATEST(length(body), 1000);
```

### 示例2：基于词汇丰富度
```sql
-- 使用唯一词比例（词汇越丰富排名越高）
UPDATE documents SET generic_rank = (
  SELECT array_length(tsvector_to_array(tsv), 1)::float / 
         NULLIF(array_length(regexp_split_to_array(body, '\s+'), 1), 0)
);
```

## 2. 内容质量指标排名

### 示例3：结合元数据
```sql
-- 综合评分、收藏数和新鲜度
UPDATE documents SET generic_rank = (
  0.4 * COALESCE(quality_score, 0.5) +
  0.3 * LOG(2 + bookmark_count) +
  0.3 * EXP(-EXTRACT(DAY FROM (NOW() - publish_date))/30)
);
```

### 示例4：作者权威性
```sql
-- 结合作者声誉
UPDATE documents d SET generic_rank = (
  SELECT 0.7 * d.quality_score + 0.3 * a.reputation_score
  FROM authors a WHERE d.author_id = a.id
);
```

## 3. 混合排名策略

### 示例5：时间衰减+热度
```sql
-- 新近且热门的文档排名高
UPDATE documents SET generic_rank = (
  LOG(2 + view_count) * 
  EXP(-EXTRACT(DAY FROM (NOW() - publish_date))/90)
);
```

### 示例6：个性化基础排名
```sql
-- 为不同类别设置不同基准
UPDATE documents SET generic_rank = CASE
  WHEN category = '新闻' THEN 0.3 * (1.0 - (NOW() - publish_date)/INTERVAL '7 days')
  WHEN category = '教程' THEN 0.7 * quality_score + 0.3 * completeness
  ELSE 0.5
END;
```

## 4. 高级文本特征排名

### 示例7：关键词集中度
```sql
-- 计算主题关键词的集中程度
UPDATE documents SET generic_rank = (
  SELECT SUM(count^2) / NULLIF(SUM(count),0)^2 
  FROM ts_stat('SELECT tsv FROM documents WHERE id = ' || id)
);
```

### 示例8：结构质量评分
```sql
-- 评估文档结构完整性
UPDATE documents SET generic_rank = (
  0.2 * CASE WHEN title IS NOT NULL THEN 1 ELSE 0 END +
  0.3 * CASE WHEN abstract IS NOT NULL THEN 1 ELSE 0 END +
  0.2 * (section_count::float / 5) +
  0.3 * (image_count::float / 3)
);
```

## 5. 实际应用场景示例

### 新闻网站通用排名
```sql
UPDATE news_articles SET generic_rank = (
  0.4 * (1.0 - (NOW() - publish_time)/INTERVAL '24 hours') + -- 时效性
  0.3 * LOG(2 + click_count) + -- 热度
  0.2 * CASE WHEN is_headline THEN 1 ELSE 0.5 END + -- 重要性
  0.1 * (1.0 / (1.0 + length(content)/5000) -- 长度适中
);
```
[新闻网站通用排名使用介绍(select)](general-rank-by-news-website/index)

### 电商产品描述通用排名
```sql
UPDATE products SET search_rank = (
  0.5 * product_rating +
  0.2 * LOG(1 + sales_count) +
  0.2 * (description_completeness / 10) +
  0.1 * CASE WHEN updated_at > NOW() - INTERVAL '7 days' THEN 1 ELSE 0.5 END
);
```

## 使用通用排名的查询示例

```sql
-- 结合通用排名和具体查询排名
SELECT id, title,
  0.4 * generic_rank + 
  0.6 * ts_rank(tsv, websearch_to_tsquery('postgresql 教程')) AS combined_rank
FROM documents
ORDER BY combined_rank DESC
LIMIT 10;

-- 作为二级排序条件
SELECT id, title, ts_rank(tsv, query) AS relevance
FROM documents, websearch_to_tsquery('数据库设计') query
WHERE tsv @@ query
ORDER BY relevance DESC, generic_rank DESC
LIMIT 20;
```

## 维护策略建议

1. **定期更新**：
   ```sql
   -- 每天更新活跃文档的通用排名
   UPDATE documents SET generic_rank = (...)
   WHERE last_activity > CURRENT_DATE - INTERVAL '7 days';
   ```

2. **增量更新**：
   ```sql
   -- 只更新变更过的文档
   UPDATE documents d SET generic_rank = (...)
   FROM changed_docs c WHERE d.id = c.id;
   ```

3. **版本控制**：
   ```sql
   ALTER TABLE documents ADD COLUMN rank_version int DEFAULT 1;
   UPDATE documents SET 
     generic_rank = (...),
     rank_version = rank_version + 1;
   ```

通用排名的关键在于找到最能反映您内容价值的指标组合，通常需要结合业务特点进行多次实验和调整才能获得最佳效果。

