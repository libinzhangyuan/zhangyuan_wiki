[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的 SEMI JOIN 和 ANTI JOIN 详解

## SEMI JOIN (半连接)

### 基本概念
SEMI JOIN（半连接）是一种特殊的连接操作，它只返回左表中那些在右表中至少有一个匹配行的行，但不会返回右表的任何数据。

### 工作原理
1. 对于左表的每一行，检查右表中是否存在匹配的行
2. 只要在右表中找到一个匹配项，左表的该行就会被包含在结果中
3. 不会重复返回左表的行（即使右表中有多个匹配）

### 在PostgreSQL中的实现方式
PostgreSQL 没有直接的 SEMI JOIN 语法，但可以通过以下方式实现：

```sql
-- 使用 EXISTS 子查询
SELECT * FROM table_a a
WHERE EXISTS (SELECT 1 FROM table_b b WHERE a.key = b.key);

-- 使用 IN 子查询
SELECT * FROM table_a a
WHERE a.key IN (SELECT b.key FROM table_b b);
```

### 执行计划表现
在 EXPLAIN 输出中，SEMI JOIN 通常显示为：
- `Hash Semi Join`
- `Merge Semi Join`
- `Nested Loop Semi Join`

### 使用场景
- 当只需要知道左表的行是否在右表中存在，而不需要右表的具体数据时
- 比常规 JOIN 更高效，因为它找到第一个匹配后就会停止搜索

## ANTI JOIN (反连接)

### 基本概念
ANTI JOIN（反连接）与 SEMI JOIN 相反，它只返回左表中那些在右表中没有匹配行的行。

### 工作原理
1. 对于左表的每一行，检查右表中是否存在匹配的行
2. 如果在右表中没有找到任何匹配项，左表的该行就会被包含在结果中

### 在PostgreSQL中的实现方式
PostgreSQL 也没有直接的 ANTI JOIN 语法，但可以通过以下方式实现：

```sql
-- 使用 NOT EXISTS 子查询
SELECT * FROM table_a a
WHERE NOT EXISTS (SELECT 1 FROM table_b b WHERE a.key = b.key);

-- 使用 NOT IN 子查询（注意NULL值问题）
SELECT * FROM table_a a
WHERE a.key NOT IN (SELECT b.key FROM table_b b WHERE b.key IS NOT NULL);
```

### 执行计划表现
在 EXPLAIN 输出中，ANTI JOIN 通常显示为：
- `Hash Anti Join`
- `Merge Anti Join`
- `Nested Loop Anti Join`

### 使用场景
- 查找不在另一个表中的记录
- 数据清洗时找出异常值或缺失值
- 实现集合差操作

## 性能考虑

1. **索引利用**：确保连接列上有适当的索引
2. **NULL值处理**：NOT IN 对NULL值敏感，NOT EXISTS 更安全
3. **数据集大小**：
   - 对于大表，Hash Semi/Anti Join 通常性能较好
   - 对于已排序的表，Merge Semi/Anti Join 可能更高效
4. **PostgreSQL优化器**：通常会将 EXISTS/NOT EXISTS 转换为高效的 Semi/Anti Join 执行计划

## 实际示例

```sql
-- SEMI JOIN 示例：找出有订单的客户
SELECT c.* FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);

-- ANTI JOIN 示例：找出没有订单的客户
SELECT c.* FROM customers c
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);
```

这两种连接操作在编写复杂查询和数据对比时非常有用，能够高效地实现"存在性检查"和"缺失性检查"的逻辑。