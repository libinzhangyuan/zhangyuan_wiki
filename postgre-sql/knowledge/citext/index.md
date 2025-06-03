[返回](/postgre-sql/knowledge/index)

# PostgreSQL 的 citext 类型介绍

citext 是 PostgreSQL 中的一个扩展数据类型，全称为 "case-insensitive text"，即不区分大小写的文本类型。它是标准 text 类型的变体，但在比较和匹配时会自动忽略大小写差异。

## 主要特点

1. **不区分大小写**：比较操作时自动忽略大小写
2. **扩展模块**：需要先启用 citext 扩展才能使用
3. **与 text 类型兼容**：可以隐式转换为 text 类型

## 安装 citext 扩展

在使用 citext 类型前，需要先安装扩展：

```sql
CREATE EXTENSION IF NOT EXISTS citext;
```

## 使用示例

### 示例1：创建使用 citext 类型的表

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username CITEXT NOT NULL,
    email CITEXT UNIQUE NOT NULL
);
```

### 示例2：插入数据并查询

```sql
INSERT INTO users (username, email) VALUES 
('JohnDoe', 'john@example.com'),
('janedoe', 'Jane@Example.com'),
('BOBSMITH', 'bob@example.com');
```

查询所有用户：

```sql
SELECT * FROM users;
```

返回结果：
```
 id | username |        email        
----+----------+--------------------
  1 | JohnDoe  | john@example.com
  2 | janedoe  | Jane@Example.com
  3 | BOBSMITH | bob@example.com
(3 rows)
```

### 示例3：不区分大小写的查询

```sql
SELECT * FROM users WHERE username = 'johndoe';
```

返回结果：
```
 id | username |       email        
----+----------+-------------------
  1 | JohnDoe  | john@example.com
(1 row)
```

### 示例4：不区分大小写的唯一约束

尝试插入大小写不同但内容相同的邮箱：

```sql
INSERT INTO users (username, email) VALUES ('johnny', 'JOHN@example.com');
```

这将违反唯一约束，因为 citext 类型认为 'john@example.com' 和 'JOHN@example.com' 是相同的。

## 与普通 text 类型的比较

```sql
-- 使用普通 text 类型
SELECT 'Text' = 'text' AS text_comparison;

-- 使用 citext 类型
SELECT 'Text'::citext = 'text'::citext AS citext_comparison;
```

返回结果：
```
 text_comparison 
-----------------
 f
(1 row)

 citext_comparison 
-------------------
 t
(1 row)
```

## 注意事项

1. citext 在比较时会自动转换为小写，因此性能可能略低于直接使用 text 类型
2. 模式匹配操作（如 LIKE 和正则表达式）默认仍是区分大小写的
3. 对于 citext 列使用 LIKE 时，可以使用 `LOWER()` 函数或 `ILIKE` 操作符

## 适用场景

citext 特别适用于需要不区分大小写的场景，如：
- 用户名、电子邮件地址的存储和比较
- 产品代码、SKU等需要忽略大小写的业务数据
- 任何需要不区分大小写唯一约束的字段

## 性能考虑

虽然 citext 提供了便利，但在大型表上可能会有性能影响，因为：
1. 比较操作需要额外的转换步骤
2. 索引大小可能会增加

在性能关键的场景中，可以考虑在应用层处理大小写转换，而不是依赖 citext 类型。