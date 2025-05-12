[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的复合类型(Composite Types)

复合类型(也称为行类型)是 PostgreSQL 中一种强大的特性，允许你创建由多个字段组成的自定义类型。

## 创建复合类型

```sql
CREATE TYPE address AS (
    street text,
    city text,
    zip_code varchar(10),
    country text
);
```

## 使用复合类型的表

```sql
CREATE TABLE customers (
    id serial PRIMARY KEY,
    name text NOT NULL,
    shipping_address address,
    billing_address address
);
```

## 插入复合类型数据

```sql
-- 使用 ROW 构造函数
INSERT INTO customers (name, shipping_address, billing_address)
VALUES (
    'John Doe',
    ROW('123 Main St', 'New York', '10001', 'USA'),
    ROW('456 Business Ave', 'New York', '10001', 'USA')
);

-- 使用显式类型转换
INSERT INTO customers (name, shipping_address)
VALUES (
    'Jane Smith',
    ('789 Park Blvd', 'Boston', '02108', 'USA')::address
);
```

## 查询复合类型数据

```sql
-- 访问整个复合字段
SELECT name, shipping_address FROM customers;

-- 访问复合字段的特定属性(使用点表示法)
SELECT name, (shipping_address).city FROM customers;

-- 使用别名简化查询
SELECT name, (sa).street, (sa).city
FROM customers, LATERAL (SELECT shipping_address AS sa) AS s;
```

## 更新复合类型

```sql
-- 更新整个复合字段
UPDATE customers
SET shipping_address = ROW('321 Oak St', 'Chicago', '60601', 'USA')
WHERE id = 1;

-- 更新复合字段的特定属性
UPDATE customers
SET shipping_address.city = 'San Francisco'
WHERE id = 2;
```

## 复合类型的高级用法

### 在函数中使用复合类型

```sql
CREATE FUNCTION get_city(addr address) RETURNS text AS $$
BEGIN
    RETURN addr.city;
END;
$$ LANGUAGE plpgsql;

SELECT name, get_city(shipping_address) FROM customers;
```

### 返回复合类型的函数

```sql
CREATE FUNCTION create_address(street text, city text, zip varchar, country text)
RETURNS address AS $$
BEGIN
    RETURN (street, city, zip, country)::address;
END;
$$ LANGUAGE plpgsql;

INSERT INTO customers (name, shipping_address)
VALUES ('Bob Brown', create_address('111 Pine St', 'Seattle', '98101', 'USA'));
```

## 复合类型的优势

1. **数据组织**：将逻辑上相关的数据组织在一起
2. **代码清晰**：提高查询和应用程序代码的可读性
3. **类型安全**：确保数据结构的完整性
4. **重用性**：可以在多个表和函数中使用相同的类型定义

## 注意事项

1. 复合类型不能有约束(如 NOT NULL)，约束需要在表级别定义
2. 修改复合类型定义需要使用 `ALTER TYPE` 命令
3. 复合类型的比较是按字段逐个比较的

复合类型是 PostgreSQL 中实现复杂数据建模的强大工具，特别适合需要将相关数据分组在一起的场景。