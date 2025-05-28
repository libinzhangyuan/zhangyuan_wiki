[返回](/postgre-sql/knowledge/index)

# PostgreSQL 常用函数大全

PostgreSQL 提供了丰富的内置函数，涵盖数学运算、字符串处理、日期时间、聚合计算等多个方面。下面我将分类介绍常用的函数及其用法示例。

## 一、数学函数

### 1. 基本数学运算

```sql
SELECT 
    ABS(-15) AS abs_value,
    CEIL(3.14) AS ceil_value,
    FLOOR(3.14) AS floor_value,
    ROUND(3.14159, 2) AS rounded_value,
    MOD(10, 3) AS remainder;
```

**返回结果：**
```
| abs_value | ceil_value | floor_value | rounded_value | remainder |
|-----------|------------|-------------|---------------|-----------|
| 15        | 4          | 3           | 3.14          | 1         |
```
### 2. 幂与对数

```sql
SELECT 
    POWER(2, 10) AS power_result,
    SQRT(25) AS sqrt_result,
    EXP(1) AS exp_result,
    LN(10) AS ln_result,
    LOG(100) AS log_result;
```

**返回结果：**
```
| power_result | sqrt_result | exp_result      | ln_result      | log_result |
|--------------|-------------|-----------------|----------------|------------|
| 1024         | 5           | 2.718281828459  | 2.302585092994 | 2          |
```
## 二、字符串函数

### 1. 字符串连接与处理

```sql
SELECT 
    CONCAT('Post', 'greSQL') AS concat_result,
    'Post' || 'greSQL' AS operator_concat,
    LENGTH('PostgreSQL') AS length_result,
    UPPER('postgresql') AS upper_result,
    LOWER('POSTGRESQL') AS lower_result;
```

**返回结果：**
```
| concat_result | operator_concat | length_result | upper_result | lower_result |
|---------------|-----------------|---------------|--------------|--------------|
| PostgreSQL    | PostgreSQL      | 10            | POSTGRESQL   | postgresql   |
```
### 2. 子字符串与位置查找

```sql
SELECT 
    SUBSTRING('PostgreSQL' FROM 5 FOR 3) AS substr_result,
    LEFT('PostgreSQL', 4) AS left_result,
    RIGHT('PostgreSQL', 6) AS right_result,
    POSITION('SQL' IN 'PostgreSQL') AS position_result,
    REPLACE('PostgreSQL', 'SQL', 'gres') AS replace_result;
```

**返回结果：**
```
| substr_result | left_result | right_result | position_result | replace_result |
|---------------|-------------|--------------|-----------------|----------------|
| gre           | Post        | tgreSQL      | 7               | Postgres       |
```
## 三、日期时间函数

### 1. 当前日期时间

```sql
SELECT 
    CURRENT_DATE AS current_date,
    CURRENT_TIME AS current_time,
    CURRENT_TIMESTAMP AS current_timestamp,
    NOW() AS now_result;
```

**返回结果示例：**
```
| current_date | current_time    | current_timestamp          | now_result                |
|--------------|-----------------|----------------------------|---------------------------|
| 2023-05-28   | 14:30:45.123456 | 2023-05-28 14:30:45.123456 | 2023-05-28 14:30:45.123456|
```
### 2. 日期运算

```sql
SELECT 
    CURRENT_DATE + INTERVAL '1 month' AS next_month,
    CURRENT_TIMESTAMP - INTERVAL '2 hours' AS two_hours_ago,
    EXTRACT(YEAR FROM CURRENT_DATE) AS current_year,
    DATE_TRUNC('month', CURRENT_DATE) AS month_start;
```

**返回结果示例：**
```
| next_month  | two_hours_ago          | current_year | month_start |
|-------------|------------------------|--------------|-------------|
| 2023-06-28  | 2023-05-28 12:30:45.123| 2023         | 2023-05-01  |
```
## 四、条件函数

### 1. CASE 表达式

```sql
SELECT 
    product_id,
    price,
    CASE 
        WHEN price > 1000 THEN '高价'
        WHEN price > 500 THEN '中价'
        ELSE '低价'
    END AS price_level
FROM products;
```

**返回结果示例：**
```
| product_id | price | price_level |
|------------|-------|-------------|
| 1          | 1200  | 高价        |
| 2          | 800   | 中价        |
| 3          | 300   | 低价        |
```
### 2. COALESCE 和 NULLIF

```sql
SELECT 
    COALESCE(NULL, NULL, 'default') AS coalesce_result,
    NULLIF(10, 10) AS nullif_equal,
    NULLIF(10, 5) AS nullif_not_equal;
```

**返回结果：**
```
| coalesce_result | nullif_equal | nullif_not_equal |
|-----------------|--------------|------------------|
| default         | NULL         | 10               |
```
## 五、聚合函数

### 1. 基本聚合

```sql
SELECT 
    COUNT(*) AS total_count,
    AVG(price) AS avg_price,
    MAX(price) AS max_price,
    MIN(price) AS min_price,
    SUM(price) AS total_price
FROM products;
```

**返回结果示例：**
```
| total_count | avg_price | max_price | min_price | total_price |
|-------------|-----------|-----------|-----------|-------------|
| 100         | 456.78    | 1999.99   | 49.99     | 45678.00    |
```
### 2. 分组聚合

```sql
SELECT 
    category,
    COUNT(*) AS count,
    STRING_AGG(product_name, ', ') AS product_list
FROM products
GROUP BY category;
```

**返回结果示例：**
```
| category | count | product_list               |
|----------|-------|----------------------------|
| 电子     | 35    | 手机, 电脑, 平板...        |
| 服装     | 45    | T恤, 牛仔裤, 外套...       |
| 食品     | 20    | 巧克力, 饼干, 饮料...      |
```
## 六、窗口函数

### 1. 排名函数

```sql
SELECT 
    product_name,
    price,
    RANK() OVER (ORDER BY price DESC) AS price_rank,
    DENSE_RANK() OVER (ORDER BY price DESC) AS dense_rank,
    ROW_NUMBER() OVER (ORDER BY price DESC) AS row_num
FROM products
LIMIT 5;
```

**返回结果示例：**
```
| product_name | price  | price_rank | dense_rank | row_num |
|--------------|--------|------------|------------|---------|
| 旗舰手机     | 1999.99| 1          | 1          | 1       |
| 高端笔记本   | 1899.99| 2          | 2          | 2       |
| 游戏本       | 1899.99| 2          | 2          | 3       |
| 平板电脑     | 1299.99| 4          | 3          | 4       |
| 智能手表     | 999.99 | 5          | 4          | 5       |
```
### 2. 聚合窗口函数

```sql
SELECT 
    product_name,
    category,
    price,
    AVG(price) OVER (PARTITION BY category) AS avg_category_price,
    price - AVG(price) OVER (PARTITION BY category) AS price_diff
FROM products
LIMIT 5;
```

**返回结果示例：**
```
| product_name | category | price  | avg_category_price | price_diff |
|--------------|----------|--------|--------------------|------------|
| 手机         | 电子     | 1999.99| 856.78             | 1143.21    |
| 电脑         | 电子     | 1899.99| 856.78             | 1043.21    |
| T恤          | 服装     | 99.99  | 145.67             | -45.68     |
| 牛仔裤       | 服装     | 199.99 | 145.67             | 54.32      |
| 巧克力       | 食品     | 49.99  | 78.90              | -28.91     |
```
## 七、JSON 函数

### 1. JSON 创建与查询

```sql
SELECT 
    JSON_BUILD_OBJECT('name', '张三', 'age', 30, 'married', true) AS json_object,
    JSON_BUILD_ARRAY(1, 2, 3, 'a', 'b', 'c') AS json_array,
    '{"name": "李四", "age": 25}'::json->>'name' AS json_value;
```

**返回结果：**
```
| json_object                          | json_array           | json_value |
|--------------------------------------|----------------------|------------|
| {"name": "张三", "age": 30, "married": true} | [1, 2, 3, "a", "b", "c"] | 李四       |
```
### 2. JSON 操作

```sql
SELECT 
    JSONB_SET('{"name": "王五", "age": 40}'::jsonb, '{age}', '45') AS updated_json,
    JSONB_DELETE('{"a": 1, "b": 2}'::jsonb, 'a') AS deleted_json,
    JSONB_PRETTY('{"name": "赵六", "details": {"age": 35, "city": "北京"}}'::jsonb) AS pretty_json;
```

**返回结果：**
```
| updated_json               | deleted_json    | pretty_json                          |
|----------------------------|-----------------|--------------------------------------|
| {"age": 45, "name": "王五"} | {"b": 2}        | {                                    |
|                            |                 |     "name": "赵六",                  |
|                            |                 |     "details": {                     |
|                            |                 |         "age": 35,                   |
|                            |                 |         "city": "北京"               |
|                            |                 |     }                                |
|                            |                 | }                                    |
```
## 八、数组函数

### 1. 数组操作

```sql
SELECT 
    ARRAY[1, 2, 3] || ARRAY[4, 5] AS array_concat,
    ARRAY_LENGTH(ARRAY['a', 'b', 'c'], 1) AS array_length,
    UNNEST(ARRAY['苹果', '香蕉', '橙子']) AS array_unnest,
    ARRAY_POSITION(ARRAY[10, 20, 30, 20], 20) AS array_position;
```

**返回结果：**
```
| array_concat | array_length | array_unnest | array_position |
|--------------|--------------|--------------|----------------|
| {1,2,3,4,5} | 3            | 苹果         | 2              |
| {1,2,3,4,5} | 3            | 香蕉         | 2              |
| {1,2,3,4,5} | 3            | 橙子         | 2              |
```
以上是 PostgreSQL 中常用的一些函数，实际应用中可以根据具体需求选择合适的函数组合使用。
