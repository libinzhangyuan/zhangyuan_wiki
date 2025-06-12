[返回](/postgre-sql/knowledge/index)

# PostgreSQL TOAST 表介绍

TOAST (The Oversized-Attribute Storage Technique) 是 PostgreSQL 用于存储大字段数据的一种技术机制。

## TOAST 基本概念

当表中存在大字段(如 text、jsonb、bytea 等)时，PostgreSQL 会使用 TOAST 技术将这些大字段数据存储在单独的 TOAST 表中，以避免主表变得过于庞大而影响性能。

### TOAST 表的特性

1. 每个可能有 TOAST 数据的表都有一个关联的 TOAST 表
2. TOAST 表与主表共享相同的 OID 命名空间，名称为 `pg_toast_<主表OID>`
3. TOAST 表只包含三个字段：chunk_id, chunk_seq 和 chunk_data
4. 数据会被压缩和/或分割成多个 chunk 存储

## 如何查看 TOAST 表

### 示例1：查看表的 TOAST 信息

```sql
SELECT relname, reltoastrelid, reltoastidxid 
FROM pg_class 
WHERE relname = 'your_table_name';
```

可能的返回结果：
```
 relname    | reltoastrelid | reltoastidxid 
------------+---------------+---------------
 your_table |         24688 |         24689
```

### 示例2：查看 TOAST 表结构

```sql
SELECT attname, atttypid::regtype 
FROM pg_attribute 
WHERE attrelid = (SELECT reltoastrelid FROM pg_class WHERE relname = 'your_table_name');
```

可能的返回结果：
```
 attname   | atttypid 
-----------+----------
 chunk_id  | oid
 chunk_seq | integer
 chunk_data| bytea
```

## TOAST 存储策略

PostgreSQL 为每个可 TOAST 的列定义了存储策略：
```
| 策略 | 描述 | SQL表示 |
|------|------|---------|
| PLAIN | 禁止压缩和行外存储 | `PLAIN` |
| EXTENDED | 允许压缩和行外存储 | `EXTENDED` |
| EXTERNAL | 允许行外存储但不压缩 | `EXTERNAL` |
| MAIN | 允许压缩但不鼓励行外存储 | `MAIN` |
```
### 示例3：查看列的 TOAST 策略

```sql
SELECT attname, attstorage 
FROM pg_attribute 
WHERE attrelid = 'your_table_name'::regclass AND attnum > 0;
```

可能的返回结果：
```
 attname  | attstorage 
----------+------------
 id       | p
 smalltxt | p
 largetxt | x
```

## TOAST 实际应用示例

### 示例4：创建一个包含 TOAST 列的表

```sql
CREATE TABLE example_toast (
    id serial PRIMARY KEY,
    small_text text,
    large_text text
);

-- 插入小数据
INSERT INTO example_toast (small_text, large_text) 
VALUES ('small data', 'this is also small data');

-- 插入大数据
INSERT INTO example_toast (small_text, large_text) 
VALUES ('small data', repeat('this is a very large text data ', 1000));
```

### 示例5：检查 TOAST 表使用情况

```sql
SELECT 
    pg_size_pretty(pg_relation_size('example_toast')) AS main_table_size,
    pg_size_pretty(pg_relation_size(pg_toast.pg_toast_table_name('example_toast'))) AS toast_table_size;
```

可能的返回结果：
```
 main_table_size | toast_table_size 
-----------------+------------------
 8192 bytes      | 0 bytes
```

## TOAST 性能考虑

1. 对于频繁访问的大字段，TOAST 可能会导致额外的 I/O 开销
2. 更新 TOAST 列时，PostgreSQL 通常会写一个新行而不是原地更新
3. 合理选择存储策略可以优化性能

### 示例6：修改列的 TOAST 策略

```sql
ALTER TABLE example_toast ALTER COLUMN large_text SET STORAGE EXTERNAL;
```

## 总结

TOAST 是 PostgreSQL 处理大字段数据的核心机制，它通过将大字段数据存储在单独的 TOAST 表中，保证了主表的高效访问。理解 TOAST 的工作原理有助于优化包含大字段的数据库设计。