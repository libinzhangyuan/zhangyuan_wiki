[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的 JSON 操作详解

PostgreSQL 提供了强大的 JSON 数据类型支持，包括 JSON 和 JSONB 两种类型。JSONB 是二进制格式，支持索引，查询效率更高，推荐使用。

## 1. 创建包含 JSON 数据的表

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    attributes JSONB,
    price NUMERIC(10,2)
);

INSERT INTO products (name, attributes, price) VALUES
('智能手机', '{"brand": "Apple", "model": "iPhone 15", "specs": {"storage": "256GB", "color": "黑色"}, "tags": ["新款", "旗舰"]}', 8999.00),
('笔记本电脑', '{"brand": "Dell", "model": "XPS 15", "specs": {"ram": "16GB", "storage": "1TB"}, "tags": ["轻薄", "高性能"]}', 12999.00),
('平板电脑', '{"brand": "Huawei", "model": "MatePad Pro", "specs": {"screen": "12.6英寸", "color": "银色"}, "tags": ["办公", "娱乐"]}', 4999.00);
```

## 2. 基本 JSON 查询操作

### 2.1 获取整个 JSON 对象

```sql
SELECT name, attributes FROM products WHERE id = 1;
```

```
 name  |                                      attributes                                      
-------+---------------------------------------------------------------------------------------
 智能手机 | {"brand": "Apple", "tags": ["新款", "旗舰"], "model": "iPhone 15", "specs": {"color": "黑色", "storage": "256GB"}}
```

### 2.2 使用 -> 操作符获取 JSON 对象字段（返回 JSON）

```sql
SELECT name, attributes->'brand' AS brand FROM products;
```

```
    name    |  brand  
------------+---------
 智能手机    | "Apple"
 笔记本电脑  | "Dell"
 平板电脑    | "Huawei"
```

### 2.3 使用 ->> 操作符获取 JSON 对象字段（返回文本）

```sql
SELECT name, attributes->>'brand' AS brand FROM products;
```

```
    name    |  brand  
------------+---------
 智能手机    | Apple
 笔记本电脑  | Dell
 平板电脑    | Huawei
```

## 3. 访问嵌套 JSON 数据

### 3.1 访问嵌套对象

```sql
SELECT name, attributes->'specs'->>'storage' AS storage FROM products;
```

```
    name    | storage 
------------+---------
 智能手机    | 256GB
 笔记本电脑  | 1TB
 平板电脑    | 
```

### 3.2 使用 #> 路径访问

```sql
SELECT name, attributes#>'{specs,color}' AS color FROM products;
```

```
    name    |  color  
------------+---------
 智能手机    | "黑色"
 笔记本电脑  | 
 平板电脑    | "银色"
```

## 4. JSON 数组操作

### 4.1 访问数组元素

```sql
SELECT name, attributes->'tags'->0 AS first_tag FROM products;
```

```
    name    | first_tag 
------------+-----------
 智能手机    | "新款"
 笔记本电脑  | "轻薄"
 平板电脑    | "办公"
```

### 4.2 检查数组中是否包含某个值

```sql
SELECT name FROM products 
WHERE attributes->'tags' ? '旗舰';
```

```
 name  
-------
 智能手机
```

## 5. JSON 函数

### 5.1 jsonb_each 展开键值对

```sql
SELECT name, jsonb_each(attributes) FROM products WHERE id = 1;
```

```
 name  |       jsonb_each       
-------+-------------------------
 智能手机 | (brand,"""Apple""")
 智能手机 | (model,"""iPhone 15""")
 智能手机 | (specs,"{""color"": ""黑色"", ""storage"": ""256GB""}")
 智能手机 | (tags,"[""新款"", ""旗舰""]")
```

### 5.2 jsonb_array_elements 展开数组

```sql
SELECT name, jsonb_array_elements(attributes->'tags') AS tag 
FROM products WHERE id = 1;
```

```
 name  |  tag  
-------+-------
 智能手机 | "新款"
 智能手机 | "旗舰"
```

## 6. JSON 条件查询

### 6.1 检查键是否存在

```sql
SELECT name FROM products 
WHERE attributes ? 'model';
```

```
    name    
------------
 智能手机
 笔记本电脑
 平板电脑
```

### 6.2 检查特定值

```sql
SELECT name FROM products 
WHERE attributes->>'brand' = 'Apple';
```

```
 name  
-------
 智能手机
```

## 7. 修改 JSON 数据

### 7.1 更新整个 JSON 对象

```sql
UPDATE products 
SET attributes = '{"brand": "Xiaomi", "model": "13 Pro", "specs": {"storage": "512GB"}}'::jsonb
WHERE id = 1;
```

### 7.2 使用 jsonb_set 更新部分字段

```sql
UPDATE products 
SET attributes = jsonb_set(attributes, '{specs,color}', '"蓝色"') 
WHERE id = 1;
```

### 7.3 添加新字段

```sql
UPDATE products 
SET attributes = jsonb_set(attributes, '{warranty}', '"2年"') 
WHERE id = 2;
```

## 8. 聚合函数与 JSON

### 8.1 jsonb_agg 将多行聚合为 JSON 数组

```sql
SELECT jsonb_agg(jsonb_build_object('name', name, 'price', price)) 
FROM products;
```

```
[{"name": "智能手机", "price": 8999.00}, {"name": "笔记本电脑", "price": 12999.00}, {"name": "平板电脑", "price": 4999.00}]
```

### 8.2 jsonb_object_agg 创建键值对对象

```sql
SELECT jsonb_object_agg(name, price) 
FROM products;
```

```
{"智能手机": 8999.00, "笔记本电脑": 12999.00, "平板电脑": 4999.00}
```

## 9. JSON 与表结构转换

### 9.1 从 JSON 构建行

```sql
SELECT * FROM jsonb_to_recordset(
  '[{"name": "耳机", "price": 599}, {"name": "鼠标", "price": 299}]'
) AS x(name text, price numeric);
```

```
 name | price 
------+-------
 耳机  |   599
 鼠标  |   299
```

PostgreSQL 的 JSON 功能非常强大，以上只是常用操作的示例。JSONB 类型还支持 GIN 索引，可以大大提高查询性能。