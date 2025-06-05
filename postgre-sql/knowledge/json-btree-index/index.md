[返回](/postgre-sql/knowledge/index)



# PostgreSQL 中的 JSON B-tree 索引

B-tree 索引是 PostgreSQL 中最常用的索引类型，也可以用于加速 JSON 数据的查询，特别是当查询针对 JSON 中的特定标量值（如字符串、数字）时。

## JSON B-tree 索引的特点

1. **高效**：对于等值查询和范围查询非常高效
2. **适用场景**：适合查询 JSON 中的特定标量值字段
3. **限制**：只能索引单个标量值，不能索引整个 JSON 文档或数组

## 创建 JSON B-tree 索引的基本语法

```sql
CREATE INDEX index_name ON table_name ((json_column->>'key'));
-- 或者对于 jsonb
CREATE INDEX index_name ON table_name ((jsonb_column->>'key'));
```

## 实际示例

### 示例表结构

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    info JSONB
);

INSERT INTO employees (info) VALUES
('{"name": "张三", "age": 28, "department": "研发", "address": {"city": "北京", "street": "中关村"}}'),
('{"name": "李四", "age": 35, "department": "销售", "address": {"city": "上海", "street": "南京路"}}'),
('{"name": "王五", "age": 42, "department": "管理", "address": {"city": "广州", "street": "天河"}}'),
('{"name": "赵六", "age": 31, "department": "研发", "address": {"city": "深圳", "street": "南山"}}');
```

### 1. 对顶级 JSON 字段创建 B-tree 索引

```sql
-- 为部门字段创建索引
CREATE INDEX idx_employee_dept ON employees ((info->>'department'));

-- 为年龄字段创建索引（注意需要转换为整数）
CREATE INDEX idx_employee_age ON employees (((info->>'age')::integer));
```

### 2. 对嵌套 JSON 字段创建 B-tree 索引

```sql
-- 为城市字段创建索引
CREATE INDEX idx_employee_city ON employees ((info#>>'{address,city}'));
```

## 使用 B-tree 索引的查询示例

### 示例 1: 查询特定部门的员工

```sql
EXPLAIN ANALYZE
SELECT info->>'name' AS name, info->>'department' AS department
FROM employees
WHERE info->>'department' = '研发';
```

返回结果：
```
| name | department |
|------|------------|
| 张三 | 研发       |
| 赵六 | 研发       |
```

执行计划会显示使用了我们创建的索引：
```
QUERY PLAN
Index Scan using idx_employee_dept on employees  (cost=0.14..8.16 rows=1 width=64)
  Index Cond: ((info ->> 'department'::text) = '研发'::text)
```

### 示例 2: 查询特定年龄范围的员工

```sql
SELECT info->>'name' AS name, (info->>'age')::int AS age
FROM employees
WHERE (info->>'age')::int BETWEEN 30 AND 40;
```

返回结果：
```
| name | age |
|------|-----|
| 李四 | 35  |
| 赵六 | 31  |
```

### 示例 3: 查询特定城市的员工

```sql
SELECT info->>'name' AS name, info#>>'{address,city}' AS city
FROM employees
WHERE info#>>'{address,city}' = '上海';
```

返回结果：
```
| name | city |
|------|------|
| 李四 | 上海 |
```

## B-tree 索引与 GIN 索引的比较
```
| 特性                | B-tree 索引                     | GIN 索引                     |
|---------------------|--------------------------------|-----------------------------|
| 适用数据类型         | 标量值（字符串、数字等）        | 整个 JSON 文档               |
| 查询类型             | 等值、范围查询                  | 包含、存在性查询             |
| 索引大小             | 较小                           | 较大                        |
| 写入性能             | 较好                           | 较差                        |
| 多值查询             | 不支持                         | 支持                        |
```
## 最佳实践建议

1. **选择合适字段**：只为频繁查询的 JSON 字段创建 B-tree 索引
2. **类型转换**：对于数值字段，确保在索引和查询中使用相同的类型转换
3. **避免过度索引**：只为真正能提升性能的查询创建索引
4. **复合索引**：可以考虑为多个 JSON 字段创建复合 B-tree 索引

```sql
-- 复合索引示例
CREATE INDEX idx_employee_dept_age ON employees (
    (info->>'department'), 
    ((info->>'age')::integer)
);
```

JSON B-tree 索引是优化特定 JSON 字段查询的有效工具，特别是在查询条件明确且针对标量值时，它能提供非常高效的查询性能。