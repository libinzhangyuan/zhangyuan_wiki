[返回](/postgre-sql/knowledge/agg-func/index)

# PostgreSQL 中的布尔聚合函数：`bool_or` 和 `bool_and`

PostgreSQL 提供了两个专门用于布尔值聚合的函数，它们在数据分析和条件筛选中非常有用。

## `bool_or` 函数

### 功能
- 对一组布尔值执行逻辑"或"（OR）运算
- 只要组内**任意一个**值为 `TRUE`，结果就是 `TRUE`
- 只有当**所有**值都是 `FALSE` 时，结果才是 `FALSE`
- 忽略 `NULL` 值（除非所有值都是 `NULL`，则返回 `NULL`）

### 语法
```sql
bool_or(boolean_expression)
```

### 使用场景
检查一组记录中**是否存在**满足特定条件的记录

### 示例
```sql
-- 检查每个部门是否有高薪员工（薪资>10万）
SELECT 
    department,
    bool_or(salary > 100000) AS has_high_paid_employee
FROM employees
GROUP BY department;

-- 筛选出有学生缺勤的班级
SELECT class_id 
FROM attendance
GROUP BY class_id
HAVING bool_or(present = false);
```

## `bool_and` 函数

### 功能
- 对一组布尔值执行逻辑"与"（AND）运算
- 只有组内**所有**值都为 `TRUE` 时，结果才是 `TRUE`
- 只要**有一个**值为 `FALSE`，结果就是 `FALSE`
- 忽略 `NULL` 值（除非所有值都是 `NULL`，则返回 `NULL`）

### 语法
```sql
bool_and(boolean_expression)
```

### 使用场景
检查一组记录是否**全部**满足特定条件

### 示例
```sql
-- 检查每个班级是否所有学生都及格（分数≥60）
SELECT 
    class_id,
    bool_and(score >= 60) AS all_passed
FROM exam_results
GROUP BY class_id;

-- 找出所有订单项都发货的订单
SELECT order_id
FROM order_items
GROUP BY order_id
HAVING bool_and(shipped = true);
```

## 对比总结
```
| 特性        | `bool_or`                     | `bool_and`                    |
|------------|------------------------------|------------------------------|
| **逻辑运算** | OR（或）                      | AND（与）                     |
| **返回TRUE** | 组内任意一个为TRUE            | 组内所有都为TRUE              |
| **返回FALSE** | 组内所有都为FALSE             | 组内任意一个为FALSE           |
| **典型用途** | 检查"是否存在"满足条件的记录   | 检查"是否全部"满足条件        |
```
## 高级用法

### 结合CASE表达式使用
```sql
-- 检查每个产品是否在所有仓库都有库存
SELECT 
    product_id,
    bool_and(CASE WHEN stock_quantity > 0 THEN TRUE ELSE FALSE END) AS all_warehouses_have_stock
FROM inventory
GROUP BY product_id;
```

### 与FILTER子句配合
```sql
-- 检查每个部门是否有研发人员且所有研发人员薪资>8万
SELECT
    department,
    bool_or(department = 'R&D') AS has_rd_staff,
    bool_and(salary > 80000) FILTER (WHERE department = 'R&D') AS all_rd_high_paid
FROM employees
GROUP BY department;
```

这两个函数在数据质量检查、业务规则验证和复杂条件筛选中特别有用，能够简化很多需要多步骤处理的查询逻辑。