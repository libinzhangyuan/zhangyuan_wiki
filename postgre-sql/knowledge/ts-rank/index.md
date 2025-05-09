[返回](/postgre-sql/knowledge/index)

# PostgreSQL 全文搜索排名函数详解

PostgreSQL 提供了两种主要的排名函数用于全文搜索结果的排序：`ts_rank()` 和 `ts_rank_cd()`。这些函数用于计算查询与文档的匹配程度，从而可以对搜索结果进行相关性排序。

## 1. ts_rank() 函数

`ts_rank()` 基于词频统计计算排名，考虑以下因素：
- 查询词在文档中出现的频率
- 查询词在文档中的位置
- 词的权重分配

### 语法
```sql
ts_rank([weights float4[],] vector tsvector, query tsquery [, normalization integer]) → float4
```

### 参数说明

1. **weights** (可选):
   - 指定四个权重类别(D, C, B, A)的权重值数组
   - 默认值: `{0.1, 0.2, 0.4, 1.0}`
   - 权重类别:
     - D: 最低权重 (默认0.1)
     - C: 次低权重 (默认0.2)
     - B: 中等权重 (默认0.4)
     - A: 最高权重 (默认1.0)

2. **vector**:
   - 文档的 `tsvector` 表示

3. **query**:
   - 搜索查询的 `tsquery` 表示

4. **normalization** (可选):
   - 控制排名归一化的位掩码
   - 可用选项(可组合使用):
     - 0 (默认): 不归一化
     - 1: 除以文档长度
     - 2: 除以唯一词的数量
     - 4: 除以平均词频
     - 8: 考虑匹配词在文档中的位置
     - 16: 考虑匹配词之间的接近度
     - 32: 考虑文档中最高权重的匹配词

### 示例

```sql
-- 基本用法
SELECT title, ts_rank(tsv, query) AS rank
FROM documents, to_tsquery('search & term') query
WHERE tsv @@ query
ORDER BY rank DESC;

-- 使用自定义权重
SELECT title, ts_rank('{0.05, 0.1, 0.2, 0.4}', tsv, query) AS rank
FROM documents, to_tsquery('search & term') query
WHERE tsv @@ query
ORDER BY rank DESC;

-- 使用归一化选项
SELECT title, ts_rank(tsv, query, 32) AS rank
FROM documents, to_tsquery('search & term') query
WHERE tsv @@ query
ORDER BY rank DESC;
```

## 2. ts_rank_cd() 函数

`ts_rank_cd()` (cover density ranking) 是 `ts_rank()` 的变体，它不仅考虑词频，还考虑匹配词之间的接近度。

### 语法
```sql
ts_rank_cd([weights float4[],] vector tsvector, query tsquery [, normalization integer]) → float4
```

### 特点
- 计算查询词在文档中的"覆盖密度"
- 对于短语搜索或需要多个词接近匹配的情况更有效
- 参数与 `ts_rank()` 相同

### 示例

```sql
-- 基本用法
SELECT title, ts_rank_cd(tsv, query) AS rank
FROM documents, to_tsquery('search <-> term') query  -- <-> 表示相邻
WHERE tsv @@ query
ORDER BY rank DESC;

-- 比较 ts_rank 和 ts_rank_cd
SELECT 
    title,
    ts_rank(tsv, query) AS rank,
    ts_rank_cd(tsv, query) AS rank_cd
FROM documents, to_tsquery('search & term') query
WHERE tsv @@ query
ORDER BY rank_cd DESC;
```

## 3. 权重分配策略

在创建 `tsvector` 时可以使用 `setweight()` 函数分配权重:

```sql
UPDATE documents SET tsv = 
    setweight(to_tsvector('english', coalesce(title,'')), 'A') || 
    setweight(to_tsvector('english', coalesce(body,'')), 'B');
```
[shangmian zhege sql jieshao (setweight desc)](setweight/index)<br>
权重分配影响排名计算，高权重的匹配会获得更高的排名分数。

## 4. [归一化选项详解](guiyihua/index)

归一化选项通过位掩码组合使用，常见组合:

```
| 值 | 描述 |
|----|------|
| 1  | 除以文档长度(词数) |
| 2  | 除以唯一词数 |
| 4  | 除以平均词频 |
| 8  | 考虑匹配词位置 |
| 16 | 考虑匹配词接近度 |
| 32 | 考虑最高权重匹配词 |

例如:
- `3` (1+2): 除以文档长度和唯一词数
- `7` (1+2+4): 除以文档长度、唯一词数和平均词频
```

## 5. 实际应用建议

1. **索引优化**:
   - 对 `tsvector` 列创建 GIN 索引
   ```sql
   CREATE INDEX tsv_idx ON documents USING gin(tsv);
   ```

2. **查询优化**:
   - 对于大型文档集，先使用 `@@` 操作符过滤，再对少量结果计算排名
   - 考虑使用 `LIMIT` 限制返回结果数量

3. **权重调整**:
   - 根据业务需求调整权重，如标题比正文更重要

4. **性能考虑**:
   - 排名计算可能消耗资源，对大型数据集要谨慎使用
   - 考虑[预计算和存储排名值](recalc-rand-and-store/index)

PostgreSQL 的排名函数提供了高度可配置的相关性排序机制，可以根据具体应用场景调整参数以获得最佳搜索体验。