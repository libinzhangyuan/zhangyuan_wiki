[返回](/postgre-sql/knowledge/index)


# PostgreSQL 整数类型 (INTEGER) 和精确数字类型 (NUMERIC) 详解

## INTEGER 类型

INTEGER 是 PostgreSQL 中的整数类型，用于存储不带小数部分的数字。

### INTEGER 类型家族

PostgreSQL 提供了多种整数类型：

```
- SMALLINT: 2字节，范围 -32,768 到 32,767
- INTEGER: 4字节，范围 -2,147,483,648 到 2,147,483,647
- BIGINT: 8字节，范围 -9,223,372,036,854,775,808 到 9,223,372,036,854,775,807
- SERIAL: 自增整数(相当于INTEGER + 序列)
- BIGSERIAL: 自增大整数(相当于BIGINT + 序列)
```

### INTEGER 示例

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    stock_quantity INTEGER,
    price NUMERIC(10,2)
);

INSERT INTO products (name, stock_quantity, price) 
VALUES ('笔记本电脑', 50, 5999.99),
       ('智能手机', 200, 2999.50),
       ('平板电脑', 75, 3999.00);
```

查询结果：

```sql
SELECT * FROM products;
```

可能的返回结果：

```
 id |     name      | stock_quantity |  price  
----+---------------+----------------+---------
  1 | 笔记本电脑    |             50 | 5999.99
  2 | 智能手机      |            200 | 2999.50
  3 | 平板电脑      |             75 | 3999.00
```

## NUMERIC 类型

NUMERIC 是 PostgreSQL 中的精确数字类型，用于存储精确的小数值，特别适合财务数据等需要精确计算的场景。

### NUMERIC 语法

```sql
NUMERIC(precision, scale)
```
- precision: 总位数
- scale: 小数位数

### NUMERIC 示例

```sql
CREATE TABLE financial_records (
    id SERIAL PRIMARY KEY,
    description VARCHAR(200),
    amount NUMERIC(15,4),
    transaction_date DATE
);

INSERT INTO financial_records (description, amount, transaction_date)
VALUES ('工资收入', 12500.5000, '2023-05-01'),
       ('房租支出', -4500.0000, '2023-05-05'),
       ('餐饮消费', -356.7500, '2023-05-10');
```

查询结果：

```sql
SELECT * FROM financial_records;
```

可能的返回结果：

```
 id |  description  |   amount   | transaction_date 
----+---------------+---------------+------------------
  1 | 工资收入      | 12500.5000  | 2023-05-01
  2 | 房租支出      | -4500.0000  | 2023-05-05
  3 | 餐饮消费      |  -356.7500  | 2023-05-10
```

## INTEGER 和 NUMERIC 的比较

### 存储范围比较

```
类型          | 存储大小 | 范围/精度
--------------+----------+----------------------------------------
SMALLINT      | 2字节    | -32,768 到 32,767
INTEGER       | 4字节    | -2,147,483,648 到 2,147,483,647
BIGINT        | 8字节    | -9,223,372,036,854,775,808 到 9,223,372,036,854,775,807
NUMERIC       | 可变     | 最高131072位前，16383位小数
```

### 使用场景比较

```
场景                | 推荐类型
--------------------+-----------
主键、外键          | INTEGER/SERIAL
计数器、数量        | INTEGER
财务数据、货币金额  | NUMERIC
科学计算            | NUMERIC
标识符、枚举值      | SMALLINT
大数据量ID          | BIGINT/BIGSERIAL
```

## 运算和函数

### INTEGER 运算

```sql
SELECT 5 / 2 AS integer_division;
```

返回结果：

```
 integer_division 
------------------
                2
```

### NUMERIC 运算

```sql
SELECT 5::NUMERIC / 2 AS numeric_division;
```

返回结果：

```
 numeric_division 
------------------
           2.5000
```

### 类型转换示例

```sql
SELECT 
    123.456::INTEGER AS to_integer,
    123::NUMERIC(5,2) AS to_numeric;
```

返回结果：

```
 to_integer | to_numeric 
------------+------------
        123 |     123.00
```

## 注意事项

1. **性能考虑**：INTEGER 运算比 NUMERIC 快，在不需要小数时应优先使用 INTEGER
2. **精度保证**：NUMERIC 提供精确计算，而浮点类型(FLOAT/DOUBLE)可能有舍入误差
3. **存储空间**：NUMERIC 是可变长度类型，可能比固定长度的整数类型占用更多空间
4. **溢出问题**：选择整数类型时要注意可能的溢出情况

## 最佳实践建议

1. 主键使用 SERIAL/BIGSERIAL
2. 财务数据使用 NUMERIC
3. 明确指定 NUMERIC 的精度和小数位数
4. 在不需要小数时使用 INTEGER 而非 NUMERIC
5. 大数据量的ID考虑使用 BIGINT