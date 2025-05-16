[返回](/postgre-sql/knowledge/index)

```
# PostgreSQL 中的 JOIN 操作详解

在 PostgreSQL 中，JOIN 操作用于将多个表中的数据组合在一起。以下是几种主要 JOIN 类型的详细讲解：

## 1. CROSS JOIN (交叉连接)

交叉连接会返回两个表的笛卡尔积，即第一个表的每一行与第二个表的每一行组合。

```sql
SELECT * FROM 表1 CROSS JOIN 表2;


SELECT * FROM 表1, 表2;
```

**特点**：
- 不需要连接条件
- 结果集行数 = 表1行数 × 表2行数
- 性能开销大，慎用

**示例**：
```sql
-- 假设表A有3行，表B有4行
SELECT * FROM A CROSS JOIN B; -- 结果将有12行
```

## 2. INNER JOIN (内连接)

内连接返回两个表中满足连接条件的行。

```sql
SELECT * FROM 表1 INNER JOIN 表2 ON 表1.列 = 表2.列;
```

**特点**：
- 只返回匹配的行
- 如果某行在一个表中没有匹配，则该行不会出现在结果中
- 最常用的 JOIN 类型

**示例**：
```sql
-- 获取有订单的客户信息
SELECT c.customer_name, o.order_date
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;
```

## 3. OUTER JOIN (外连接)

外连接返回一个表的所有行，即使在另一个表中没有匹配。

### 3.1 LEFT OUTER JOIN (左外连接)
返回左表的所有行，右表没有匹配则显示 NULL。

```sql
SELECT * FROM 表1 LEFT OUTER JOIN 表2 ON 表1.列 = 表2.列;
```

### 3.2 RIGHT OUTER JOIN (右外连接)
返回右表的所有行，左表没有匹配则显示 NULL。

```sql
SELECT * FROM 表1 RIGHT OUTER JOIN 表2 ON 表1.列 = 表2.列;
```

### 3.3 FULL OUTER JOIN (全外连接)
返回两个表的所有行，没有匹配的部分显示 NULL。

```sql
SELECT * FROM 表1 FULL OUTER JOIN 表2 ON 表1.列 = 表2.列;
```

**示例**：
```sql
-- 获取所有客户及其订单(包括没有订单的客户)
SELECT c.customer_name, o.order_date
FROM customers c
LEFT OUTER JOIN orders o ON c.customer_id = o.customer_id;

-- 获取所有订单及客户信息(包括没有客户的订单)
SELECT c.customer_name, o.order_date
FROM customers c
RIGHT OUTER JOIN orders o ON c.customer_id = o.customer_id;

-- 获取所有客户和所有订单的完整信息
SELECT c.customer_name, o.order_date
FROM customers c
FULL OUTER JOIN orders o ON c.customer_id = o.customer_id;
```

## 总结比较
```
| JOIN 类型       | 描述                                                                 |
|----------------|--------------------------------------------------------------------|
| CROSS JOIN     | 返回两个表的笛卡尔积，无匹配条件                                         |
| INNER JOIN     | 只返回两个表中匹配的行                                                 |
| LEFT JOIN      | 返回左表所有行，右表不匹配则为 NULL                                      |
| RIGHT JOIN     | 返回右表所有行，左表不匹配则为 NULL                                      |
| FULL JOIN      | 返回两个表的所有行，不匹配的部分为 NULL                                   |
```
在实际应用中，INNER JOIN 和 LEFT JOIN 是最常用的两种连接方式。


```