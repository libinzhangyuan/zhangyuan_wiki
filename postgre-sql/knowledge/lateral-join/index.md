[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的 LATERAL JOIN 详解

LATERAL JOIN 是 PostgreSQL 中一种特殊的连接方式，它允许右侧的子查询引用左侧表的列，从而实现行级的关联查询。

## 基本概念

LATERAL JOIN 的关键特点是可以让右侧的子查询或函数引用左侧表当前行的列值，相当于为左侧表的每一行执行一次右侧的查询。

## 语法格式

```sql
SELECT 列列表
FROM 左表
[LEFT | INNER] JOIN LATERAL (
  子查询
) 别名 ON 条件;
```

## 使用场景

### 1. 与子查询一起使用

```sql
-- 为每个用户获取他们最近的3个订单
SELECT u.user_id, u.name, o.order_id, o.order_date
FROM users u
LEFT JOIN LATERAL (
  SELECT * 
  FROM orders 
  WHERE user_id = u.user_id 
  ORDER BY order_date DESC 
  LIMIT 3
) o ON true;
```

### 2. 与函数一起使用

```sql
-- 为每个用户生成指定数量的随机数
SELECT u.user_id, u.name, r.random_num
FROM users u
CROSS JOIN LATERAL generate_series(1, u.num_count) AS r(random_num);
```

## 与普通 JOIN 的区别

1. **引用能力**：LATERAL JOIN 允许右侧查询引用左侧表的列，而普通 JOIN 不行
2. **执行顺序**：LATERAL JOIN 先处理左侧表的行，再为每行执行右侧查询
3. **结果集**：右侧查询的结果可以与左侧表的当前行相关联

## 常见应用

1. **分页查询**：为每行获取关联表的前N条记录
2. **JSON/数组展开**：展开JSON或数组字段并与原表关联
3. **行生成**：基于每行的值生成多行数据

## 性能考虑

- LATERAL JOIN 可能比普通 JOIN 更消耗资源，因为它要为左侧表的每一行执行右侧查询
- 当左侧表行数多时，性能影响显著
- 合理使用索引可以提高 LATERAL JOIN 的性能

## 示例

```sql
-- 展开JSON数组并与原表关联
SELECT o.order_id, o.order_date, i.item_name
FROM orders o
JOIN LATERAL jsonb_array_elements(o.items) AS i(item_name) ON true;

-- 为每个部门获取薪水最高的3名员工
SELECT d.dept_name, e.emp_name, e.salary
FROM departments d
LEFT JOIN LATERAL (
  SELECT emp_name, salary
  FROM employees
  WHERE dept_id = d.dept_id
  ORDER BY salary DESC
  LIMIT 3
) e ON true;
```

LATERAL JOIN 是 PostgreSQL 强大的功能之一，特别适合处理需要行级关联查询的复杂场景。