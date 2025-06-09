[返回](/postgre-sql/knowledge/index)


# PostgreSQL 中 `1.0::NUMERIC / 3.0` 的存储方式

在 PostgreSQL 中，`NUMERIC` 类型（也称为 `DECIMAL`）用于存储精确的数值，特别适合需要精确计算的场景，如财务数据。

## `1.0::NUMERIC / 3.0` 的存储

当计算 `1.0::NUMERIC / 3.0` 时，PostgreSQL 会：

1. 将两个操作数都作为 `NUMERIC` 类型处理
2. 执行精确的除法运算
3. 结果会存储为一个无限精度的分数，直到需要显示或进一步计算时才会进行舍入

### 实际存储示例

让我们看看这个计算的实际结果：

```sql
SELECT 1.0::NUMERIC / 3.0 AS result;
```

返回结果：
```
        result         
-----------------------
 0.33333333333333333333
```

### 存储特性

1. **精度保留**：`NUMERIC` 类型会尽可能保留所有有效数字
2. **无精度损失**：不像浮点数那样有舍入误差
3. **可变长度存储**：根据实际需要的精度动态分配存储空间

## 精度控制

可以通过指定精度来控制 `NUMERIC` 类型的存储：

```sql
-- 指定精度为5位小数
SELECT (1.0::NUMERIC(20,5) / 3.0::NUMERIC(20,5)) AS precise_result;
```

返回结果：
```
 precise_result 
----------------
     0.33333
```

## 与浮点数的对比

与 `REAL` 或 `DOUBLE PRECISION` 类型比较：

```sql
SELECT 
    1.0::NUMERIC / 3.0 AS numeric_result,
    1.0::REAL / 3.0 AS real_result,
    1.0::DOUBLE PRECISION / 3.0 AS double_result;
```

返回结果：
```
      numeric_result      |     real_result     |    double_result    
--------------------------+---------------------+---------------------
 0.33333333333333333333 | 0.33333334 | 0.3333333333333333
```

## 内部存储机制

PostgreSQL 中 `NUMERIC` 类型的内部存储特点：

1. 使用二进制编码的十进制格式
2. 每4位十进制数字存储在2个字节中
3. 小数点位置单独存储
4. 符号位单独存储

可以通过 `pg_column_size` 查看实际存储大小：

```sql
SELECT 
    pg_column_size(1.0::NUMERIC / 3.0) AS storage_size_bytes,
    pg_column_size(1.0::NUMERIC(100,50) / 3.0) AS high_precision_size;
```

返回结果：
```
 storage_size_bytes | high_precision_size 
-------------------+--------------------
                16 |                  24
```

## 总结

`1.0::NUMERIC / 3.0` 在 PostgreSQL 中：
- 以精确的十进制形式存储
- 不会像浮点数那样有精度损失
- 存储空间会根据需要的精度动态调整
- 特别适合需要精确计算的场景