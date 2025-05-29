[返回](/postgre-sql/knowledge/index)


# PostgreSQL 排名函数比较：row_number vs rank vs dense_rank

PostgreSQL 提供了三种常用的窗口排名函数：`row_number()`、`rank()` 和 `dense_rank()`。它们都用于为结果集中的行分配排名，但在处理相同值(ties)时有不同的行为。

## 三种函数的对比
```
| 函数 | 特点 | 相同值的处理 | 序号是否连续 |
|------|------|--------------|--------------|
| `row_number()` | 为每一行分配唯一序号 | 相同值也会分配不同序号 | 总是连续 |
| `rank()` | 相同值分配相同排名 | 下一个不同值会跳过序号 | 可能不连续 |
| `dense_rank()` | 相同值分配相同排名 | 下一个不同值不会跳过序号 | 总是连续 |
```
## 详细说明与示例

### 1. row_number()

- **特点**：为每一行分配唯一的连续序号，从1开始
- **相同值处理**：即使值相同，也会分配不同的序号

```sql
SELECT 
    student_name,
    score,
    row_number() OVER (ORDER BY score DESC) as row_num
FROM test_scores;
```

示例结果：
```
 student_name | score | row_num 
--------------+-------+---------
 张三         |  95   |    1
 李四         |  90   |    2
 王五         |  90   |    3
 赵六         |  85   |    4
```

### 2. rank()

- **特点**：相同值获得相同排名，后续排名会跳过序号
- **相同值处理**：相同值获得相同排名，下一个不同值会跳过序号

```sql
SELECT 
    student_name,
    score,
    rank() OVER (ORDER BY score DESC) as rank
FROM test_scores;
```

示例结果：
```
 student_name | score | rank 
--------------+-------+------
 张三         |  95   |   1
 李四         |  90   |   2
 王五         |  90   |   2
 赵六         |  85   |   4  -- 注意这里跳过了3
```

### 3. dense_rank()

- **特点**：相同值获得相同排名，但后续排名不会跳过序号
- **相同值处理**：相同值获得相同排名，下一个不同值序号连续

```sql
SELECT 
    student_name,
    score,
    dense_rank() OVER (ORDER BY score DESC) as dense_rank
FROM test_scores;
```

示例结果：
```
 student_name | score | dense_rank 
--------------+-------+------------
 张三         |  95   |     1
 李四         |  90   |     2
 王五         |  90   |     2
 赵六         |  85   |     3  -- 注意这里没有跳过序号
```

## 综合比较示例

```sql
SELECT 
    student_name,
    score,
    row_number() OVER (ORDER BY score DESC) as row_num,
    rank() OVER (ORDER BY score DESC) as rank,
    dense_rank() OVER (ORDER BY score DESC) as dense_rank
FROM test_scores
ORDER BY score DESC;
```

示例结果：
```
 student_name | score | row_num | rank | dense_rank 
--------------+-------+---------+------+------------
 张三         |  95   |    1    |  1   |     1
 李四         |  90   |    2    |  2   |     2
 王五         |  90   |    3    |  2   |     2
 赵六         |  85   |    4    |  4   |     3
 钱七         |  80   |    5    |  5   |     4
 孙八         |  80   |    6    |  5   |     4
 周九         |  75   |    7    |  7   |     5
```

## 实际应用场景

1. **row_number()**：
   - 需要绝对唯一的行标识时
   - 实现分页查询
   - 删除重复数据(保留每组中的第一条)

2. **rank()**：
   - 体育比赛排名(允许并列，后续排名跳过)
   - 学术成绩排名

3. **dense_rank()**：
   - 需要连续排名的场景
   - 等级划分(如金、银、铜牌)

## 分区使用示例

所有三个函数都可以与 `PARTITION BY` 一起使用：

```sql
SELECT 
    department,
    employee_name,
    salary,
    row_number() OVER (PARTITION BY department ORDER BY salary DESC) as dept_row_num,
    rank() OVER (PARTITION BY department ORDER BY salary DESC) as dept_rank,
    dense_rank() OVER (PARTITION BY department ORDER BY salary DESC) as dept_dense_rank
FROM employees;
```

示例结果：
```
 department | employee_name | salary | dept_row_num | dept_rank | dept_dense_rank 
------------+---------------+--------+--------------+-----------+-----------------
 技术部     | 王五          | 9500   |      1       |     1     |        1
 技术部     | 张三          | 8000   |      2       |     2     |        2
 技术部     | 李四          | 8000   |      3       |     2     |        2
 销售部     | 赵六          | 7500   |      1       |     1     |        1
 销售部     | 钱七          | 7000   |      2       |     2     |        2
 销售部     | 孙八          | 6500   |      3       |     3     |        3
```