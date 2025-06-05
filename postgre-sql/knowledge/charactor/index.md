[返回](/postgre-sql/knowledge/index)

# PostgreSQL 字符类型详解

PostgreSQL 提供了多种字符类型来存储文本数据，每种类型有不同的特点和适用场景。

## 主要字符类型

```
类型              | 别名         | 描述
------------------+--------------+-------------------------------------------------
CHAR(n)          | CHARACTER(n) | 固定长度，空格填充，最多1GB
VARCHAR(n)       | CHARACTER VARYING(n) | 可变长度，有长度限制，最多1GB
TEXT             | -            | 可变长度，无长度限制，最多1GB
"char"           | -            | 单字节字符类型，内部使用
name             | -            | 内部用于标识符，63字节定长
```

## 类型详细说明

### 1. CHAR(n) - 定长字符串

- 固定长度，不足部分用空格填充
- 存储时总是占用n个字节
- 最大长度1GB

**示例：**
```sql
CREATE TABLE char_example (
    id SERIAL PRIMARY KEY,
    country_code CHAR(2),
    description CHAR(10)
);

INSERT INTO char_example (country_code, description)
VALUES ('CN', 'China'), ('US', 'USA'), ('JP', 'Japan');
```

**查询结果：**
```sql
SELECT id, country_code, description, length(description) 
FROM char_example;
```
```
 id | country_code | description | length 
----+--------------+-------------+--------
  1 | CN           | China       |      5
  2 | US           | USA         |      3
  3 | JP           | Japan       |      5
```

### 2. VARCHAR(n) - 变长字符串

- 可变长度，只占用实际需要的空间
- 可以指定最大长度限制
- 最大长度1GB

**示例：**
```sql
CREATE TABLE varchar_example (
    id SERIAL PRIMARY KEY,
    username VARCHAR(20),
    bio VARCHAR(500)
);

INSERT INTO varchar_example (username, bio)
VALUES 
    ('user1', 'Software developer from Beijing'),
    ('user2', 'Designer');
```

**查询结果：**
```sql
SELECT id, username, length(bio) AS bio_length 
FROM varchar_example;
```
```
 id | username | bio_length 
----+----------+------------
  1 | user1    |         30
  2 | user2    |          7
```

### 3. TEXT - 无限长文本

- 可变长度，无长度限制（最多1GB）
- 性能通常优于VARCHAR
- PostgreSQL推荐用于一般文本存储

**示例：**
```sql
CREATE TABLE text_example (
    id SERIAL PRIMARY KEY,
    title VARCHAR(100),
    content TEXT
);

INSERT INTO text_example (title, content)
VALUES 
    ('PostgreSQL', 'PostgreSQL is a powerful, open source object-relational database system...'),
    ('TEXT Type', 'The TEXT data type can store unlimited length of characters...');
```

**查询结果：**
```sql
SELECT id, title, length(content) AS content_length 
FROM text_example;
```
```
 id |    title    | content_length 
----+-------------+----------------
  1 | PostgreSQL  |             71
  2 | TEXT Type   |             63
```

## 类型比较

```
特性                | CHAR(n)         | VARCHAR(n)       | TEXT
-------------------+-----------------+------------------+-----------------
存储方式            | 定长，空格填充  | 变长             | 变长
是否限制长度        | 是              | 是               | 否
存储效率            | 低              | 高               | 高
典型用途            | 固定长度代码    | 有限长度字符串    | 任意长度文本
最大长度            | 1GB             | 1GB              | 1GB
索引性能            | 一般            | 好               | 好
```

## 特殊字符类型

### 1. "char" (带引号的char)

- 单字节字符类型
- 主要用于系统目录
- 不是CHAR的简写

**示例：**
```sql
SELECT 'a'::"char" AS single_char;
```
```
 single_char 
-------------
 a
```

### 2. name 类型

- 内部用于系统标识符
- 63字节定长
- 通常不建议用户使用

## 字符串函数示例

### 1. 连接字符串

```sql
SELECT first_name || ' ' || last_name AS full_name 
FROM customers;
```

### 2. 子字符串

```sql
SELECT substring('PostgreSQL' FROM 6 FOR 3) AS part;
```
```
 part 
------
 SQL
```

### 3. 转换大小写

```sql
SELECT lower('PostgreSQL'), upper('PostgreSQL');
```
```
   lower    |   upper   
-----------+-----------
 postgresql| POSTGRESQL
```

### 4. 模式匹配

```sql
SELECT 'Hello' LIKE 'He%' AS matches;
```
```
 matches 
---------
 t
```

## 最佳实践

1. **优先使用TEXT**：PostgreSQL官方推荐，除非需要长度约束
2. **避免CHAR**：除非确实需要固定长度，否则浪费空间
3. **合理使用VARCHAR长度**：设置合理的最大长度作为约束
4. **注意编码**：确保数据库编码(如UTF-8)支持所需字符
5. **索引考虑**：长文本列考虑使用表达式索引或全文搜索

## 性能考虑

1. TEXT和VARCHAR性能差异很小
2. 过长的字符串可能影响排序和索引性能
3. 模式匹配(LIKE)在长文本上较慢
4. 考虑使用`text_pattern_ops`索引加速模式匹配

```sql
CREATE INDEX idx_name ON users (last_name text_pattern_ops);
```