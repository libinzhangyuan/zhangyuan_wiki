[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的 IS DISTINCT FROM 运算符

`IS DISTINCT FROM` 是 PostgreSQL 中的一个比较运算符，用于比较两个值是否不同，与普通的 `!=` 或 `<>` 运算符不同，它能够正确处理 NULL 值。

## 主要特点

1. 当比较两个非 NULL 值时，`IS DISTINCT FROM` 的行为与 `!=` 相同
2. 当比较 NULL 值时，它能正确处理 NULL 与 NULL 的比较
3. 返回布尔值 TRUE 或 FALSE

## 与普通比较运算符的区别

普通比较运算符在遇到 NULL 时总是返回 NULL，而 `IS DISTINCT FROM` 会返回明确的 TRUE 或 FALSE。

## 语法

```sql
expression IS DISTINCT FROM expression
```

## 示例

### 示例1：比较非NULL值

```sql
SELECT 
    1 IS DISTINCT FROM 2 AS "1 ≠ 2",
    1 IS DISTINCT FROM 1 AS "1 ≠ 1",
    'a' IS DISTINCT FROM 'b' AS "'a' ≠ 'b'";
```

返回结果：
```
 1 ≠ 2 | 1 ≠ 1 | 'a' ≠ 'b' 
-------+-------+-----------
 t     | f     | t
```

### 示例2：比较NULL值

```sql
SELECT 
    NULL IS DISTINCT FROM NULL AS "NULL ≠ NULL",
    NULL IS DISTINCT FROM 1 AS "NULL ≠ 1",
    1 IS DISTINCT FROM NULL AS "1 ≠ NULL";
```

返回结果：
```
 NULL ≠ NULL | NULL ≠ 1 | 1 ≠ NULL 
-------------+----------+----------
 f           | t        | t
```

### 示例3：实际数据查询

假设有一个员工表 employees:

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(100),
    salary INT
);

INSERT INTO employees (name, department, salary) VALUES
('张三', '销售部', 5000),
('李四', NULL, 6000),
('王五', '技术部', NULL),
('赵六', '技术部', 7000);
```

查询部门不为'技术部'或部门为NULL的员工：

```sql
SELECT name, department 
FROM employees 
WHERE department IS DISTINCT FROM '技术部';
```

返回结果：
```
 name | department 
------+------------
 张三 | 销售部
 李四 | NULL
```

### 示例4：与普通比较运算符对比

```sql
SELECT 
    (NULL = NULL) AS "NULL = NULL",
    (NULL != NULL) AS "NULL != NULL",
    (NULL IS NOT DISTINCT FROM NULL) AS "NULL IS NOT DISTINCT FROM NULL",
    (NULL IS DISTINCT FROM NULL) AS "NULL IS DISTINCT FROM NULL";
```

返回结果：
```
 NULL = NULL | NULL != NULL | NULL IS NOT DISTINCT FROM NULL | NULL IS DISTINCT FROM NULL 
-------------+--------------+--------------------------------+----------------------------
 NULL        | NULL         | t                              | f
```

## 使用场景

1. 当需要比较可能包含NULL值的列时
2. 在WHERE子句中需要明确区分NULL和非NULL值时
3. 在CHECK约束中处理NULL值比较

## 注意事项

1. `IS DISTINCT FROM` 是标准SQL的一部分，不是PostgreSQL特有的
2. 对应的否定形式是 `IS NOT DISTINCT FROM`
3. 在索引使用上，`IS DISTINCT FROM` 可能不如普通比较运算符高效