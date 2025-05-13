[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的函数索引(Functional Index)

函数索引是基于表达式或函数计算结果创建的索引，而不是直接基于列值。这种索引特别适用于需要对数据进行转换或计算后再查询的场景。

## 函数索引基础

### 基本语法

```sql
CREATE INDEX index_name ON table_name (expression);
```

### 简单示例

```sql
-- 对用户名的小写形式创建索引
CREATE INDEX idx_users_lower_username ON users (lower(username));

-- 现在可以使用索引加速不区分大小写的查询
SELECT * FROM users WHERE lower(username) = 'alice';
```

## 常见使用场景

### 1. 不区分大小写搜索

```sql
CREATE INDEX idx_products_lower_name ON products (lower(product_name));

-- 使用索引的查询
SELECT * FROM products WHERE lower(product_name) = 'laptop';
```

### 2. 日期部分查询

```sql
-- 只索引日期的年份部分
CREATE INDEX idx_orders_year ON orders (extract(year from order_date));

-- 查询特定年份的订单
SELECT * FROM orders WHERE extract(year from order_date) = 2023;
```

### 3. JSON数据查询

```sql
-- 索引JSON字段中的特定属性
CREATE INDEX idx_properties_json ON properties ((data->>'price')::numeric);

-- 查询JSON属性
SELECT * FROM properties WHERE (data->>'price')::numeric > 100000;
```

### 4. 计算列索引

```sql
-- 基于计算结果的索引
CREATE INDEX idx_employees_salary_bonus ON employees (salary + bonus);

-- 查询总报酬
SELECT * FROM employees WHERE salary + bonus > 100000;
```

## 高级用法

### 1. 条件函数索引(部分索引)

```sql
-- 只为活跃用户创建索引
CREATE INDEX idx_active_users_email ON users (lower(email)) 
WHERE is_active = true;

-- 只查询活跃用户时会使用索引
SELECT * FROM users WHERE is_active = true AND lower(email) = 'user@example.com';
```

### 2. 多列函数索引

```sql
-- 组合多个列的索引
CREATE INDEX idx_full_name ON employees (lower(first_name || ' ' || last_name));

-- 查询全名
SELECT * FROM employees 
WHERE lower(first_name || ' ' || last_name) = 'john smith';
```

### 3. 使用内置函数

```sql
-- 使用trim和lower函数
CREATE INDEX idx_clean_address ON customers (trim(lower(address)));

-- 查询清理后的地址
SELECT * FROM customers WHERE trim(lower(address)) = '123 main st';
```

## 函数索引的注意事项

1. **维护成本**：函数索引会增加写操作的开销
2. **精确匹配**：查询必须使用与索引定义完全相同的表达式
3. **不可变函数**：索引表达式必须使用不可变函数(immutable)
4. **统计信息**：可能需要手动ANALYZE表以更新统计信息

## 查看函数索引

```sql
-- 查看所有函数索引
SELECT 
    indexname AS index_name,
    indexdef AS definition
FROM 
    pg_indexes
WHERE 
    indexdef LIKE '%(%' 
    AND schemaname = 'public';
```

## 管理函数索引

### 重建函数索引

```sql
REINDEX INDEX idx_users_lower_username;
```

### 删除函数索引

```sql
DROP INDEX IF EXISTS idx_users_lower_username;
```

## 性能考虑

1. **选择性**：确保函数索引有足够的选择性
2. **使用频率**：只为频繁使用的查询模式创建
3. **测试验证**：使用EXPLAIN验证索引是否被使用
4. **替代方案**：考虑使用计算列+普通索引作为替代

函数索引是PostgreSQL中强大的功能，可以显著优化特定查询模式的性能，但需要谨慎使用以避免不必要的维护开销。