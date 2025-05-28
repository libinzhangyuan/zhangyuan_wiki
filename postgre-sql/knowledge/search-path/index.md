[返回](/postgre-sql/knowledge/index)

# PostgreSQL 默认 Schema 设置介绍

在 PostgreSQL 中，Schema 是一个命名空间，包含表、视图、索引等数据库对象。设置默认 Schema 可以让你在查询时不必每次都指定 Schema 名称。

## 查看当前 Schema

```sql
SELECT current_schema();
```

```
 current_schema 
----------------
 public
(1 row)
```

## 设置默认 Schema 的方法

### 1. 使用 SET 命令临时设置

```sql
SET search_path TO myschema, public;
```

这个设置只在当前会话有效。

### 2. 修改用户级默认 Schema

```sql
ALTER USER your_username SET search_path TO myschema, public;
```

### 3. 修改数据库级默认 Schema

```sql
ALTER DATABASE your_database SET search_path TO myschema, public;
```

## 示例操作

### 创建新 Schema

```sql
CREATE SCHEMA myschema;
```

### 在新 Schema 中创建表

```sql
CREATE TABLE myschema.employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(50)
);
```

### 插入数据

```sql
INSERT INTO myschema.employees (name, department) 
VALUES ('张三', '技术部'), ('李四', '市场部');
```

### 查询数据（设置默认 Schema 前）

```sql
SELECT * FROM myschema.employees;
```

```
 id | name | department 
----+------+------------
  1 | 张三 | 技术部
  2 | 李四 | 市场部
(2 rows)
```

### 设置默认 Schema 后查询

```sql
SET search_path TO myschema, public;
SELECT * FROM employees;
```

```
 id | name | department 
----+------+------------
  1 | 张三 | 技术部
  2 | 李四 | 市场部
(2 rows)
```

## 查看当前搜索路径

```sql
SHOW search_path;
```

```
   search_path   
-----------------
 myschema, public
(1 row)
```

## 注意事项

1. 搜索路径中的第一个 Schema 是创建新对象的默认位置
2. `public` Schema 是 PostgreSQL 的默认 Schema
3. 当对象名称相同时，搜索路径中靠前的 Schema 优先

通过合理设置默认 Schema，可以使 SQL 语句更简洁，并实现多租户等架构设计。