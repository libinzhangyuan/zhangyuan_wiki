[返回](/postgre-sql/knowledge/index)

# PostgreSQL 的 ILIKE 操作符详解

ILIKE 是 PostgreSQL 中用于进行不区分大小写的模式匹配的操作符，它是 LIKE 操作符的不区分大小写版本。

## 基本语法

```sql
expression ILIKE pattern [ESCAPE escape_character]
```

## ILIKE 与 LIKE 的区别

- `LIKE` 是区分大小写的
- `ILIKE` 是不区分大小写的（仅PostgreSQL支持，不是SQL标准）

## 通配符说明

- `%` - 匹配任意数量的字符（包括零个字符）
- `_` - 匹配单个字符

## 示例

### 示例1：基本不区分大小写匹配

```sql
SELECT 'PostgreSQL' ILIKE 'postgresql';
```

```
 ?column? 
----------
 t
```

### 示例2：在表中使用 ILIKE

假设有一个用户表 `users`:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

INSERT INTO users (name, email) VALUES 
('张三', 'zhangsan@example.com'),
('李四', 'lisi@example.com'),
('王五', 'wangwu@example.com'),
('John Doe', 'john.doe@example.com'),
('Jane Smith', 'jane.smith@example.com');
```

查询名字中包含 "zhang" 的用户（不区分大小写）：

```sql
SELECT * FROM users WHERE name ILIKE '%zhang%';
```

```
 id | name |         email         
----+------+-----------------------
  1 | 张三 | zhangsan@example.com
```

### 示例3：使用通配符

查找邮箱以 "JANE" 开头的用户（不区分大小写）：

```sql
SELECT * FROM users WHERE email ILIKE 'jane%';
```

```
 id |    name    |         email         
----+------------+-----------------------
  5 | Jane Smith | jane.smith@example.com
```

### 示例4：混合使用通配符

查找名字第二个字母是 "o" 的用户：

```sql
SELECT * FROM users WHERE name ILIKE '_o%';
```

```
 id |   name   |        email        
----+----------+---------------------
  4 | John Doe | john.doe@example.com
```

## ILIKE 与索引

需要注意的是，ILIKE 通常无法使用普通 B-tree 索引。如果需要优化 ILIKE 查询性能，可以考虑：

1. 使用 `citext` 扩展的数据类型
2. 创建表达式索引：`CREATE INDEX idx_name_lower ON users (LOWER(name))`

## 性能考虑

ILIKE 比 LIKE 性能稍差，因为它需要进行大小写不敏感的匹配。在大表上使用时应注意性能影响。

## 总结

PostgreSQL 的 ILIKE 操作符提供了方便的不区分大小写的模式匹配功能，特别适合需要忽略大小写的文本搜索场景。