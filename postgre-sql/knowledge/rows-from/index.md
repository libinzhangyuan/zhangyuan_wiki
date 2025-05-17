[返回](/postgre-sql/knowledge/index)


# PostgreSQL 中的 ROWS FROM 语法介绍

在 PostgreSQL 中，`ROWS FROM` 是一个用于表函数(table functions)的特殊语法，它允许你同时调用多个函数并将它们的结果作为一组列返回。

## 基本语法

```sql
ROWS FROM(function_name(args), function_name(args), ...) [AS] alias(column_name, ...)
```

## 功能特点

1. **多函数组合**：可以同时调用多个函数，每个函数返回的结果集将被并排组合
2. **列对齐**：如果函数返回的行数不同，较短的结果集会用 NULL 值填充以匹配最长结果集的行数
3. **列命名**：可以通过 AS 子句为结果列指定名称

## 使用示例

### 基本用法

```sql
SELECT * FROM ROWS FROM(generate_series(1,3), generate_series(5,7)) AS t(a, b);
```

结果：
```
 a | b
---+---
 1 | 5
 2 | 6
 3 | 7
```

### 不同行数的函数组合

```sql
SELECT * FROM ROWS FROM(generate_series(1,3), generate_series(5,6)) AS t(a, b);
```

结果：
```
 a | b
---+---
 1 | 5
 2 | 6
 3 | NULL
```

### 与 LATERAL 结合使用

```sql
SELECT * FROM 
  (VALUES (1,5), (2,6), (3,7)) AS x(a,b),
  LATERAL ROWS FROM(generate_series(a,a+1), generate_series(b,b+1)) AS t(c,d);
```

## 应用场景

1. 需要同时调用多个集合返回函数并合并结果
2. 需要将多个函数的输出进行列式组合
3. 在复杂查询中简化多个函数调用的语法

`ROWS FROM` 是 PostgreSQL 特有的语法，提供了比标准 SQL 更灵活的函数结果处理方式。