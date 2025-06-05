[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的 NaN 和 Infinity 处理

在 PostgreSQL 中，NaN (Not a Number) 和 Infinity (无穷大) 是特殊的浮点数值，主要出现在 `real` 和 `double precision` 数据类型中。

## NaN (非数字)

NaN 表示一个未定义或不可表示的数值结果。

### 示例

```sql
SELECT 'NaN'::real AS nan_value;
```

```
 nan_value 
-----------
       NaN
```

## Infinity (无穷大)

PostgreSQL 支持正无穷大和负无穷大：

### 示例

```sql
SELECT 'Infinity'::double precision AS pos_infinity, 
       '-Infinity'::double precision AS neg_infinity;
```

```
 pos_infinity | neg_infinity 
--------------+--------------
  Infinity    | -Infinity
```

## 特殊值的生成

这些值可以通过计算或直接转换得到：

```sql
SELECT 0.0/0.0 AS nan_example, 
       1.0/0.0 AS pos_inf_example, 
       -1.0/0.0 AS neg_inf_example;
```

```
 nan_example | pos_inf_example | neg_inf_example 
-------------+-----------------+-----------------
         NaN |         Infinity |      -Infinity
```

## 检测特殊值

PostgreSQL 提供了特殊函数来检测这些值：

```sql
SELECT 
  val,
  val = 'NaN' AS eq_nan,
  isfinite(val) AS is_finite,
  val = 'Infinity' AS eq_inf,
  val = '-Infinity' AS eq_neg_inf
FROM (VALUES (1.0), ('NaN'::real), ('Infinity'::real), ('-Infinity'::real)) AS t(val);
```

```
    val    | eq_nan | is_finite | eq_inf | eq_neg_inf 
-----------+--------+-----------+--------+-----------
         1 | f      | t         | f      | f
       NaN | t      | f         | f      | f
 Infinity  | f      | f         | t      | f
 -Infinity | f      | f         | f      | t
```

## 排序行为

这些特殊值在排序时有特定行为：

```sql
SELECT val 
FROM (VALUES (1.0), ('NaN'::real), ('Infinity'::real), ('-Infinity'::real)) AS t(val)
ORDER BY val;
```

```
    val    
-----------
 -Infinity
         1
 Infinity
       NaN
```

## 数学运算

与这些特殊值的运算结果：

```sql
SELECT 
  'NaN'::real + 1 AS nan_plus_1,
  'Infinity'::real + 1 AS inf_plus_1,
  'Infinity'::real * 0 AS inf_times_0;
```

```
 nan_plus_1 | inf_plus_1 | inf_times_0 
------------+------------+-------------
        NaN |  Infinity  |         NaN
```

## 注意事项

1. NaN 不等于任何值，包括它自己
2. 使用 `IS DISTINCT FROM` 可以正确处理 NaN 的比较
3. 聚合函数通常忽略 NaN 值

```sql
SELECT 
  'NaN'::real = 'NaN'::real AS nan_eq_nan,
  'NaN'::real IS DISTINCT FROM 'NaN'::real AS nan_is_distinct;
```

```
 nan_eq_nan | nan_is_distinct 
------------+-----------------
 f          | f
```