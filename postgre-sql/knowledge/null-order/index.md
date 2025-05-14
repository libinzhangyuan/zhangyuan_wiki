[返回](/postgre-sql/knowledge/index)

# PostgreSQL 索引中的 NULL 值排序

在 PostgreSQL 中，索引对 NULL 值的处理方式可以通过特定语法进行控制，这在创建索引时非常重要，特别是当查询需要以特定方式处理 NULL 值时。

## 默认行为

默认情况下，PostgreSQL 的 B-tree 索引会包含 NULL 值，但它们的排序位置取决于索引类型和查询条件：

- **普通索引**：NULL 值会被包含在索引中，但排序位置不确定
- **唯一索引**：允许存在多个 NULL 值（因为 NULL 不等于 NULL）

## 显式控制 NULL 排序

PostgreSQL 允许在创建索引时显式指定 NULL 值的排序位置：

### 1. NULLS FIRST (NULL 值排在前面)

```sql
CREATE INDEX idx_name ON table_name (column_name NULLS FIRST);
```

### 2. NULLS LAST (NULL 值排在后面)

```sql
CREATE INDEX idx_name ON table_name (column_name NULLS LAST);
```

## 使用场景

### 场景1：ORDER BY 查询优化

```sql
-- 创建与查询排序一致的索引
CREATE INDEX idx_employees_hire_date ON employees (hire_date NULLS LAST);

-- 优化这样的查询
SELECT * FROM employees ORDER BY hire_date NULLS LAST;
```

### 场景2：复合索引中的 NULL 处理

```sql
CREATE INDEX idx_orders ON orders (customer_id NULLS LAST, order_date);
```

### 场景3：唯一索引中的 NULL 处理

```sql
-- 允许 email 为 NULL，但非 NULL 值必须唯一
CREATE UNIQUE INDEX idx_users_email ON users (email) WHERE email IS NOT NULL;
-- 或者
CREATE UNIQUE INDEX idx_users_email ON users (email NULLS LAST);
```

## 实际示例

### 示例1：销售数据报表

```sql
-- 折扣可能为 NULL（表示无折扣）
CREATE INDEX idx_products_discount ON products (discount NULLS LAST);

-- 获取按折扣排序的产品，无折扣产品排在最后
SELECT product_name, price, discount 
FROM products 
ORDER BY discount NULLS LAST;
```

### 示例2：员工入职日期

```sql
-- 新员工可能还没有入职日期（NULL）
CREATE INDEX idx_employees_hire_date ON employees (hire_date NULLS FIRST);

-- 找出尚未入职的员工（会使用索引）
SELECT * FROM employees 
WHERE hire_date IS NULL;
```

## 性能考虑

1. **查询与索引匹配**：确保 `ORDER BY` 子句中的 NULL 排序与索引定义一致才能充分利用索引
2. **索引大小**：包含 NULL 值的索引会稍大，但通常影响不大
3. **部分索引**：如果不关心 NULL 值，可以创建过滤掉 NULL 的部分索引
   ```sql
   CREATE INDEX idx_active_users ON users (last_login) 
   WHERE last_login IS NOT NULL;
   ```

## 查看索引的 NULL 排序

```sql
SELECT pg_get_indexdef(indexrelid) 
FROM pg_index 
WHERE indrelid = 'table_name'::regclass;
```

## 最佳实践

1. **一致性**：使索引的 NULL 排序与常用查询的排序一致
2. **明确性**：总是显式指定 NULL 排序，避免依赖默认行为
3. **查询分析**：使用 `EXPLAIN ANALYZE` 验证索引是否被正确使用
4. **部分索引**：对于很少查询 NULL 值的情况，考虑使用部分索引排除 NULL

通过合理控制索引中的 NULL 值排序，可以显著提高包含 NULL 值排序查询的性能。










# 跟mysql不一样

# PostgreSQL 与 MySQL 中 NULL 值排序对比

NULL 值在数据库中的排序行为是一个重要的实现差异，PostgreSQL 和 MySQL 在这方面有显著不同。以下是两者的详细对比：

## 1. 默认排序行为

### PostgreSQL
- **默认排序**：`NULLS FIRST` (NULL 值排在最前面)
- **升序排序(ASC)**：`NULLS FIRST`
- **降序排序(DESC)**：`NULLS LAST`
- **可显式控制**：可以使用 `NULLS FIRST` 或 `NULLS LAST` 明确指定

```sql
-- PostgreSQL 默认
SELECT * FROM table ORDER BY column; -- NULL 在前
SELECT * FROM table ORDER BY column DESC; -- NULL 在后
```

### MySQL
- **默认排序**：`NULLS LAST` (NULL 值排在最后面)
- **升序排序(ASC)**：`NULLS LAST`
- **降序排序(DESC)**：`NULLS FIRST`
- **无法显式控制**：没有提供 `NULLS FIRST/LAST` 语法

```sql
-- MySQL 默认
SELECT * FROM table ORDER BY column; -- NULL 在后
SELECT * FROM table ORDER BY column DESC; -- NULL 在前
```

## 2. 索引中的 NULL 处理

### PostgreSQL
- 可以在创建索引时指定 NULL 的排序位置
- 支持 `NULLS FIRST` 和 `NULLS LAST` 语法
- 对查询优化器更透明

```sql
CREATE INDEX idx_name ON table(column NULLS FIRST);
CREATE INDEX idx_name ON table(column NULLS LAST);
```

### MySQL
- 没有显式的索引 NULL 排序控制
- NULL 值总是被放在索引的最后（对于升序索引）
- 实现方式不如 PostgreSQL 灵活

## 3. 唯一索引中的 NULL 值

### PostgreSQL
- 唯一索引允许包含多个 NULL 值（因为 NULL ≠ NULL）
- 可以通过部分索引限制 NULL 值数量

```sql
CREATE UNIQUE INDEX idx_unique ON table(column) WHERE column IS NOT NULL;
```

### MySQL
- 行为取决于存储引擎：
  - **InnoDB**：允许多个 NULL 值
  - **MyISAM**：允许多个 NULL 值
- 与 PostgreSQL 类似，但实现细节不同

## 4. 比较运算符中的 NULL

两者在比较运算中的行为相同：
- 任何与 NULL 的比较都返回 UNKNOWN (被当作 FALSE 处理)
- 必须使用 `IS NULL` 或 `IS NOT NULL` 判断 NULL 值

## 5. 实际影响示例

### 场景：员工表按入职日期排序

**PostgreSQL 实现**：
```sql
-- 默认NULL在前
SELECT name, hire_date FROM employees ORDER BY hire_date;

-- 明确指定NULL在后
SELECT name, hire_date FROM employees ORDER BY hire_date NULLS LAST;
```

**MySQL 实现**：
```sql
-- 默认NULL在后
SELECT name, hire_date FROM employees ORDER BY hire_date;

-- 要实现NULL在前，需要复杂写法
SELECT name, hire_date FROM employees 
ORDER BY CASE WHEN hire_date IS NULL THEN 0 ELSE 1 END, hire_date;
```

## 6. 迁移注意事项

当从MySQL迁移到PostgreSQL或反之时，关于NULL排序需要注意：

1. **应用逻辑**：检查依赖NULL排序位置的业务逻辑
2. **查询重写**：可能需要重写ORDER BY子句
3. **索引优化**：在PostgreSQL中可以更精细地控制索引中的NULL排序
4. **分页查询**：分页结果可能因NULL排序不同而不同

## 总结对比表
```
| 特性                | PostgreSQL                          | MySQL                            |
|---------------------|------------------------------------|----------------------------------|
| 默认ORDER BY ASC    | NULLS FIRST                        | NULLS LAST                      |
| 默认ORDER BY DESC   | NULLS LAST                         | NULLS FIRST                     |
| 显式NULL排序控制    | 支持(NULLS FIRST/LAST)             | 不支持                          |
| 索引中的NULL控制    | 创建索引时可指定                   | 不可指定                        |
| 唯一索引中的NULL    | 允许多个NULL                       | 依存储引擎而定(通常允许多个NULL) |
| 实现灵活性          | 更高                               | 较低                            |
```
PostgreSQL 提供了更强大和灵活的 NULL 值排序控制，而 MySQL 的实现相对简单但不够灵活。在设计跨数据库应用时，这种差异需要特别注意。