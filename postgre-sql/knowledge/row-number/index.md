[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的 row_number() 函数详解

`row_number()` 是 PostgreSQL 中的一个窗口函数，它为结果集中的每一行分配一个唯一的序号，从 1 开始依次递增。

## 基本语法

```sql
row_number() OVER (
    [PARTITION BY partition_expression, ... ]
    ORDER BY sort_expression [ASC | DESC], ...
)
```

## 主要特点

1. 为每一行分配唯一的连续整数
2. 序号从 1 开始
3. 通常与 `OVER()` 子句一起使用
4. 可以按分区(`PARTITION BY`)和排序(`ORDER BY`)来组织数据

## 使用示例

### 示例1：基本用法

```sql
SELECT 
    id, 
    name, 
    salary,
    row_number() OVER (ORDER BY salary DESC) as rank
FROM employees;
```

可能的返回结果：

```
 id |   name   | salary | rank 
----+----------+--------+------
  3 | 王五     |  9500  |  1
  1 | 张三     |  8000  |  2
  4 | 赵六     |  7500  |  3
  2 | 李四     |  7000  |  4
```

### 示例2：带分区的 row_number

```sql
SELECT 
    department, 
    name, 
    salary,
    row_number() OVER (PARTITION BY department ORDER BY salary DESC) as dept_rank
FROM employees;
```

可能的返回结果：

```
 department |  name  | salary | dept_rank 
------------+--------+--------+-----------
 技术部     | 王五   |  9500  |     1
 技术部     | 张三   |  8000  |     2
 销售部     | 赵六   |  7500  |     1
 销售部     | 李四   |  7000  |     2
```

### 示例3：使用 row_number 分页

```sql
-- 获取第2页数据，每页3条
SELECT * FROM (
    SELECT 
        id, 
        name, 
        salary,
        row_number() OVER (ORDER BY id) as row_num
    FROM employees
) AS numbered_rows
WHERE row_num BETWEEN 4 AND 6;
```

可能的返回结果：

```
 id |  name  | salary | row_num 
----+--------+--------+---------
  4 | 赵六   |  7500  |    4
  5 | 钱七   |  6800  |    5
  6 | 孙八   |  7200  |    6
```

### 示例4：删除重复数据

```sql
-- 删除重复数据，只保留每组中 row_number=1 的记录
DELETE FROM duplicates
WHERE id IN (
    SELECT id FROM (
        SELECT 
            id,
            row_number() OVER (PARTITION BY column1, column2 ORDER BY id) as rn
        FROM duplicates
    ) t
    WHERE t.rn > 1
);
```

## 注意事项

1. `row_number()` 分配的序号是临时的，不会存储在表中
2. 与 `rank()` 和 `dense_rank()` 不同，`row_number()` 总是生成连续的唯一序号，即使有相同值的行
3. 性能考虑：在大数据集上使用窗口函数可能会有性能开销

## 与其他排名函数的比较

```sql
SELECT 
    name,
    score,
    row_number() OVER (ORDER BY score DESC) as row_num,
    rank() OVER (ORDER BY score DESC) as rank,
    dense_rank() OVER (ORDER BY score DESC) as dense_rank
FROM students;
```

可能的返回结果：

```
 name  | score | row_num | rank | dense_rank 
-------+-------+---------+------+------------
 张三  |  95   |    1    |  1   |     1
 李四  |  90   |    2    |  2   |     2
 王五  |  90   |    3    |  2   |     2
 赵六  |  85   |    4    |  4   |     3
```