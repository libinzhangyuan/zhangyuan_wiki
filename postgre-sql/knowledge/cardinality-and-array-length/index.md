[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的 `cardinality` 和 `array_length` 函数比较

这两个函数都用于处理数组，但功能有所不同：

## `array_length(arr, dim)`

**功能**：返回数组在指定维度上的长度

**特点**：
- 需要指定维度参数（从1开始）
- 只返回指定维度的长度
- 对于多维数组，可以分别查询每一维的长度
- 如果数组为NULL或指定维度不存在，返回NULL

**示例**：
```sql
SELECT array_length(ARRAY[[1,2],[3,4],[5,6]], 1);  -- 返回 3（第一维有3个子数组）
SELECT array_length(ARRAY[[1,2],[3,4],[5,6]], 2);  -- 返回 2（每个子数组有2个元素）
```

## `cardinality(arr)`

**功能**：返回数组中所有元素的总数（所有维度的元素总和）

**特点**：
- 不需要指定维度
- 计算数组中所有元素的个数，包括所有嵌套维度
- 如果数组为NULL，返回NULL

**示例**：
```sql
SELECT cardinality(ARRAY[[1,2],[3,4],[5,6]]);  -- 返回 6（总共6个元素）
SELECT cardinality(ARRAY[1,2,3]);  -- 返回 3
```

## 主要区别
```
| 特性        | array_length | cardinality |
|-----------|-------------|-------------|
| 返回值      | 指定维度的长度 | 所有元素总数 |
| 参数        | 需要维度参数   | 不需要维度参数 |
| 多维数组处理 | 可查询特定维度 | 返回所有维度元素总和 |
| NULL处理   | 返回NULL     | 返回NULL     |
```
## 使用场景选择

- 当需要知道数组的"形状"（各维度大小）时，使用`array_length`
- 当需要知道数组中元素的总数时，使用`cardinality`
- 对于一维数组，`array_length(arr, 1)`和`cardinality(arr)`返回相同结果