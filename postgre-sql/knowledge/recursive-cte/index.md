[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的递归 CTE (Common Table Expression)

递归 CTE 是 PostgreSQL 中一种强大的查询技术，允许您处理层次结构或树形数据，执行迭代计算，以及解决需要多次自引用的问题。

## 基本语法

递归 CTE 由两部分组成，通过 `WITH RECURSIVE` 关键字定义：

```sql
WITH RECURSIVE cte_name AS (
    -- 非递归部分 (基础查询)
    SELECT ... FROM ... WHERE ...
    
    UNION [ALL]
    
    -- 递归部分 (引用CTE自身的查询)
    SELECT ... FROM ... JOIN cte_name ON ...
)
SELECT * FROM cte_name;
```

## 组成部分详解

1. **非递归部分**：提供递归的初始结果集（种子查询）
2. **UNION/UNION ALL**：连接非递归和递归部分
   - `UNION` 会去重
   - `UNION ALL` 保留所有行，性能更好
3. **递归部分**：引用 CTE 自身，产生新的结果

## 实际应用示例

### 1. 计算数字序列

生成1到10的数字序列：

```sql
WITH RECURSIVE numbers AS (
    SELECT 1 AS n  -- 基础查询
    
    UNION ALL
    
    SELECT n + 1   -- 递归部分
    FROM numbers
    WHERE n < 10   -- 终止条件
)
SELECT * FROM numbers;
```

### 2. 遍历树形结构（员工层级）

假设有员工表 `employees(id, name, manager_id)`：

```sql
WITH RECURSIVE employee_hierarchy AS (
    -- 基础查询：顶级管理者(manager_id为NULL的员工)
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- 递归查询：下属员工
    SELECT e.id, e.name, e.manager_id, eh.level + 1
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.id
)
SELECT * FROM employee_hierarchy ORDER BY level, id;
```

### 3. 查找路径（图遍历）

查找所有从A到D的路径：

```sql
WITH RECURSIVE paths AS (
    -- 基础查询：所有起点为A的边
    SELECT start_node, end_node, ARRAY[start_node, end_node] AS path
    FROM edges
    WHERE start_node = 'A'
    
    UNION ALL
    
    -- 递归查询：扩展路径
    SELECT p.start_node, e.end_node, p.path || e.end_node
    FROM paths p
    JOIN edges e ON p.end_node = e.start_node
    WHERE NOT e.end_node = ANY(p.path)  -- 避免循环
)
SELECT * FROM paths WHERE end_node = 'D';
```

## 重要注意事项

1. **终止条件**：递归部分必须包含终止条件，否则会无限循环
2. **性能考虑**：递归CTE可能消耗大量资源，对大表要谨慎使用
3. **循环检测**：处理图数据时需防止无限循环
4. **深度限制**：PostgreSQL默认限制递归深度为1000，可通过`SET max_recursive_iterations = N`调整

## 高级用法

### 1. 累积计算

计算斐波那契数列：

```sql
WITH RECURSIVE fibonacci(n, a, b) AS (
    SELECT 1, 0, 1
    
    UNION ALL
    
    SELECT n + 1, b, a + b
    FROM fibonacci
    WHERE n < 10
)
SELECT a FROM fibonacci;
```

### 2. 带聚合的递归查询

计算组织总薪资：

```sql
WITH RECURSIVE org_salary AS (
    -- 基础查询：所有部门
    SELECT id, name, parent_id, salary
    FROM departments
    WHERE parent_id IS NULL
    
    UNION ALL
    
    -- 递归查询：累加子部门薪资
    SELECT d.id, d.name, d.parent_id, d.salary + os.salary
    FROM departments d
    JOIN org_salary os ON d.parent_id = os.id
)
SELECT * FROM org_salary;
```

递归CTE是PostgreSQL中处理层次结构和图数据的强大工具，合理使用可以解决许多复杂的查询问题。
