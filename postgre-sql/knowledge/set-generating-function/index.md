[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的集合生成函数 (Set Generating Functions)

在 PostgreSQL 中，集合生成函数(Set Generating Functions)是一类特殊的函数，它们能够返回多行数据而不是单个值。这些函数在查询中非常有用，特别是当你需要将单个行"展开"为多行时。

## 主要集合生成函数

### 1. `generate_series()`

这是最常用的集合生成函数，用于生成一系列值：

```sql
-- 生成数字序列
SELECT * FROM generate_series(1, 5);
-- 结果: 1, 2, 3, 4, 5

-- 带步长的序列
SELECT * FROM generate_series(1, 10, 2);
-- 结果: 1, 3, 5, 7, 9

-- 生成日期序列
SELECT * FROM generate_series(
    '2023-01-01'::timestamp,
    '2023-01-05'::timestamp,
    '1 day'
);
```

### 2. `unnest()`

将数组展开为多行：

```sql
SELECT unnest(ARRAY['a', 'b', 'c']);
-- 结果: a, b, c
```

### 3. `regexp_split_to_table()`

按正则表达式拆分字符串并返回多行：

```sql
SELECT regexp_split_to_table('hello,world,postgres', ',');
-- 结果: hello, world, postgres
```

## 使用场景

1. **数据生成**：快速创建测试数据
2. **时间序列分析**：填充缺失的日期
3. **数组处理**：将数组元素转换为行
4. **字符串处理**：拆分字符串为多行

## 高级用法

集合生成函数可以与其他SQL功能结合使用：

```sql
-- 与LATERAL结合使用
SELECT t.id, s.*
FROM my_table t,
LATERAL generate_series(1, t.count) s;

-- 在CTE中使用
WITH dates AS (
  SELECT generate_series(
    current_date,
    current_date + '1 week'::interval,
    '1 day'::interval
  ) AS date
)
SELECT * FROM dates;
```

集合生成函数是PostgreSQL强大功能的重要组成部分，它们大大扩展了SQL的表达能力。