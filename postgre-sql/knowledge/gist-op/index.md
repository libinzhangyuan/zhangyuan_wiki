[返回](/postgre-sql/knowledge/index)

# PostgreSQL GIST 索引自定义操作符类

PostgreSQL 的 GIST (Generalized Search Tree) 索引支持多种自定义操作符类(operator classes)，这些操作符类定义了索引如何与特定数据类型和操作符交互。以下是主要的 GIST 自定义操作符类：

## 内置 GIST 操作符类

### 1. 几何类型操作符类

```sql
-- 点类型操作符
CREATE INDEX idx_point ON points_table USING gist(point_column);

-- 支持的操作符示例
-- <<  严格在左侧
-- >>  严格在右侧
-- &<  不超过右侧
-- &>  不超过左侧
-- <-> 两点间距离
```

### 2. 范围类型操作符类

```sql
-- 范围类型操作符
CREATE INDEX idx_range ON ranges_table USING gist(range_column);

-- 支持的操作符示例
-- &&  重叠
-- @>  包含
-- <@  被包含
-- <<  严格在左侧
-- >>  严格在右侧
-- &<  不超过右侧
-- &>  不超过左侧
-- -|- 相邻
```

### 3. 网络地址类型操作符类

```sql
-- inet/cidr 类型操作符
CREATE INDEX idx_inet ON network_table USING gist(ip_column inet_ops);

-- 支持的操作符示例
-- <<  包含于
-- <<= 包含于或等于
-- >>  包含
-- >>= 包含或等于
-- &&  包含或被包含
```

### 4. 全文搜索操作符类

```sql
-- 文本搜索向量操作符
CREATE INDEX idx_tsvector ON docs_table USING gist(tsvector_column);

-- 支持的操作符示例
-- @@  文本搜索匹配
```

### 5. 数组类型操作符类

```sql
-- 数组重叠操作符
CREATE INDEX idx_array ON array_table USING gist(array_column array_ops);

-- 支持的操作符示例
-- &&  数组重叠
```

## 自定义 GIST 操作符类示例

### 1. 创建自定义类型和操作符类

```sql
-- 创建自定义数据类型
CREATE TYPE complex_number AS (
    real_part float8,
    imag_part float8
);

-- 创建比较函数
CREATE FUNCTION complex_abs(complex_number) RETURNS float8 AS $$
    SELECT sqrt($1.real_part^2 + $1.imag_part^2);
$$ LANGUAGE SQL IMMUTABLE;

-- 创建支持函数
CREATE FUNCTION complex_gist_consistent(internal, complex_number, smallint, oid, internal)
RETURNS bool AS 'MODULE_PATHNAME' LANGUAGE C;

-- 创建操作符类
CREATE OPERATOR CLASS complex_ops
    DEFAULT FOR TYPE complex_number USING gist AS
        OPERATOR        1       < ,
        OPERATOR        2       <= ,
        OPERATOR        3       = ,
        OPERATOR        4       >= ,
        OPERATOR        5       > ,
        FUNCTION        1       complex_gist_consistent (internal, complex_number, smallint, oid, internal),
        FUNCTION        2       complex_abs(complex_number);
```

### 2. 使用自定义操作符类

```sql
-- 创建使用自定义操作符类的索引
CREATE TABLE complex_numbers (
    id serial PRIMARY KEY,
    value complex_number
);

CREATE INDEX idx_complex ON complex_numbers USING gist(value complex_ops);
```

## 查看现有操作符类

```sql
-- 查看所有GIST操作符类
SELECT opcname, opcintype::regtype, opcdefault
FROM pg_opclass
WHERE opcmethod = (SELECT oid FROM pg_am WHERE amname = 'gist')
ORDER BY opcname;
```

可能的返回结果：

```
        opcname        |       opcintype       | opcdefault 
------------------------|-----------------------|------------
 array_ops             | anyarray              | t
 box_ops               | box                   | t
 circle_ops            | circle                | t
 inet_ops              | inet                  | t
 point_ops             | point                 | t
 polygon_ops           | polygon               | t
 range_ops             | anyrange              | t
 tsquery_ops           | tsquery               | t
 tsvector_ops          | tsvector              | t
 complex_ops           | complex_number        | t
```

## 常用 GIST 操作符类支持的操作符

以下是常见 GIST 操作符类支持的主要操作符：

```
-- 几何类型常见操作符
<<  -- 严格在左侧
>>  -- 严格在右侧
&<  -- 不超过右侧
&>  -- 不超过左侧
<-> -- 距离计算
@>  -- 包含
<@  -- 被包含
&&  -- 重叠

-- 范围类型常见操作符
&&  -- 重叠
@>  -- 包含元素
@>  -- 包含范围
<@  -- 被范围包含
<<  -- 严格在左侧
>>  -- 严格在右侧
&<  -- 不超过右侧
&>  -- 不超过左侧
-|- -- 相邻

-- 网络地址常见操作符
<<   -- 包含于
<<=  -- 包含于或等于
>>   -- 包含
>>=  -- 包含或等于
&&   -- 包含或被包含
```

GIST 操作符类的灵活性使得它可以为各种复杂数据类型提供高效的索引支持，特别是那些具有空间或层次结构特性的数据。