[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中 NULL 值的管理

在 PostgreSQL 中，NULL 是一个特殊值，表示缺失、未知或不适用的数据。正确处理 NULL 值是数据库管理的重要部分。

## NULL 的基本特性

1. **比较运算**：任何与 NULL 的比较都返回 NULL（未知）
   ```sql
   SELECT 1 = NULL;  -- 返回 NULL
   SELECT 1 != NULL; -- 返回 NULL
   ```

2. **IS NULL 和 IS NOT NULL**：专门用于检查 NULL 值
   ```sql
   SELECT * FROM table WHERE column IS NULL;
   SELECT * FROM table WHERE column IS NOT NULL;
   ```

## 处理 NULL 的函数

1. **COALESCE**：返回第一个非 NULL 的参数
   ```sql
   SELECT COALESCE(NULL, NULL, 'default'); -- 返回 'default'
   ```

2. **NULLIF**：如果两个参数相等则返回 NULL，否则返回第一个参数
   ```sql
   SELECT NULLIF(5, 5); -- 返回 NULL
   SELECT NULLIF(5, 10); -- 返回 5
   ```

3. **GREATEST/LEAST**：会忽略 NULL 值
   ```sql
   SELECT GREATEST(1, NULL, 3); -- 返回 3
   ```

## 聚合函数与 NULL

大多数聚合函数（如 SUM, AVG）会忽略 NULL 值：
```sql
SELECT AVG(column) FROM table; -- 自动忽略 NULL 值
```

COUNT 函数的不同行为：
```sql
SELECT COUNT(*) FROM table; -- 计算所有行，包括 NULL
SELECT COUNT(column) FROM table; -- 只计算非 NULL 值
```

## 排序中的 NULL

在排序时，NULL 默认被视为大于所有非 NULL 值：
```sql
SELECT * FROM table ORDER BY column; -- NULL 值排在最后
```

可以使用 NULLS FIRST 或 NULLS LAST 改变行为：
```sql
SELECT * FROM table ORDER BY column NULLS FIRST;
```

## 索引与 NULL

默认情况下，B-tree 索引不包含 NULL 值。如果需要索引 NULL 值，可以使用：
```sql
CREATE INDEX idx_name ON table (column) WHERE column IS NULL;
```

## 最佳实践

1. 明确区分 NULL 和空字符串/零值
2. 在应用程序中始终考虑 NULL 的可能性
3. 使用 COALESCE 提供默认值
4. 在 WHERE 子句中正确使用 IS NULL/IS NOT NULL

理解并正确管理 NULL 值对于编写可靠的 PostgreSQL 查询至关重要。