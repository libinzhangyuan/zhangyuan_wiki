[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的自增主键 ID

在 PostgreSQL 中，自增主键 ID 是一种常用的方式来为表中的每行数据自动生成唯一的标识符。以下是关于 PostgreSQL 自增主键的详细介绍：

## 实现方式

PostgreSQL 提供了几种方法来实现自增主键：

### 1. SERIAL 类型

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);
```

`SERIAL` 是 PostgreSQL 的便捷写法，实际上它会创建一个整数列和一个关联的序列(sequence)。

### 2. IDENTITY 列 (PostgreSQL 10+)

```sql
CREATE TABLE users (
    id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(100)
);
```

这是 SQL 标准语法，PostgreSQL 10 及以上版本支持。

### 3. 显式使用序列(SEQUENCE)

```sql
CREATE SEQUENCE users_id_seq;
CREATE TABLE users (
    id INTEGER NOT NULL DEFAULT nextval('users_id_seq') PRIMARY KEY,
    name VARCHAR(100)
);
```

## 特点

1. **自动递增**：每次插入新行时，ID 会自动增加
2. **唯一性**：保证每行都有唯一的标识符
3. **不可为空**：作为主键，ID 列不允许 NULL 值
4. **高效查询**：主键自动创建索引，提高查询效率

## 注意事项

- 使用 `SERIAL` 时，PostgreSQL 会自动创建名为 `表名_列名_seq` 的序列
- 大表考虑使用 `BIGSERIAL` 代替 `SERIAL`，以避免整数溢出
- 在分布式系统中，自增 ID 可能不是最佳选择，考虑 UUID 或其他方案
- 使用 `RETURNING` 子句可以在插入后立即获取生成的 ID

## 示例插入

```sql
INSERT INTO users (name) VALUES ('张三') RETURNING id;
```

这将插入新行并返回自动生成的 ID 值。