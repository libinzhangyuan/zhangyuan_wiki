[返回](/postgre-sql/knowledge/index)

# PostgreSQL 索引类型: GIST 和 SPGIST 介绍

## GIST (Generalized Search Tree) 索引

GIST 是 PostgreSQL 中的一种通用搜索树索引，它可以用于实现多种不同的搜索策略。GIST 索引特别适合用于复杂数据类型(如几何类型、全文搜索等)和自定义数据类型。

### GIST 特点
- 支持多种数据类型
- 可以自定义操作符类
- 适用于"包含"、"相交"等关系查询
- 支持 kNN (k-nearest-neighbor) 搜索

### GIST 示例

#### 1. 创建几何数据表并添加 GIST 索引

```sql
CREATE TABLE spatial_data (
    id serial PRIMARY KEY,
    name varchar(100),
    geom geometry(Point, 4326)
);

CREATE INDEX idx_spatial_data_geom ON spatial_data USING GIST(geom);
```

#### 2. 查询附近点

```sql
-- 插入测试数据
INSERT INTO spatial_data (name, geom) VALUES 
('地点1', ST_SetSRID(ST_MakePoint(116.404, 39.915), 4326)),
('地点2', ST_SetSRID(ST_MakePoint(116.405, 39.916), 4326)),
('地点3', ST_SetSRID(ST_MakePoint(116.406, 39.917), 4326));

-- 查询距离 (116.404, 39.915) 1000米内的点
SELECT name, ST_AsText(geom) 
FROM spatial_data 
WHERE ST_DWithin(
    geom, 
    ST_SetSRID(ST_MakePoint(116.404, 39.915), 4326)::geography, 
    1000
);
```

返回结果:
```
name  |       st_astext       
------+-----------------------
地点1 | POINT(116.404 39.915)
地点2 | POINT(116.405 39.916)
```

## SPGIST (Space-Partitioned Generalized Search Tree) 索引

SPGIST 是 PostgreSQL 9.2 引入的一种索引类型，它使用空间分区的方法来组织数据，特别适合用于高度不规则的数据分布。

### SPGIST 特点
- 适合高度不规则的数据分布
- 对于某些查询模式比 GIST 更高效
- 支持点类型、范围类型和某些文本搜索操作
- 可以避免 GIST 在某些情况下的性能问题

### SPGIST 示例

#### 1. 创建 IP 地址表并使用 SPGIST 索引

```sql
CREATE TABLE ip_addresses (
    id serial PRIMARY KEY,
    ip inet,
    description text
);

CREATE INDEX idx_ip_addresses_ip ON ip_addresses USING SPGIST(ip);
```

#### 2. 查询包含在某个网络中的 IP

```sql
-- 插入测试数据
INSERT INTO ip_addresses (ip, description) VALUES 
('192.168.1.1', '内部服务器1'),
('192.168.1.2', '内部服务器2'),
('10.0.0.1', '外部服务器1');

-- 查询 192.168.1.0/24 网络中的所有 IP
SELECT ip, description 
FROM ip_addresses 
WHERE ip << inet '192.168.1.0/24';
```

返回结果:
```
     ip      |   description    
-------------+------------------
 192.168.1.1 | 内部服务器1
 192.168.1.2 | 内部服务器2
```

## GIST 和 SPGIST 对比

```
| 特性                | GIST                          | SPGIST                        |
|---------------------|-------------------------------|-------------------------------|
| 适用数据类型        | 多种复杂数据类型              | 点、范围、某些文本类型        |
| 数据分布适应性      | 适合规则分布                  | 适合高度不规则分布            |
| 索引大小            | 通常较大                      | 通常较小                      |
| 查询性能            | 范围查询快                    | 点查询快                      |
| 支持的操作          | 包含、相交、距离等            | 包含、相邻等                  |
| 版本支持            | PostgreSQL 所有版本           | PostgreSQL 9.2+               |
| 最适合场景          | 地理数据、全文搜索            | IP地址、不规则分布的点数据    |
```

## 选择建议

1. 对于地理空间数据，通常 GIST 是更好的选择
2. 对于 IP 地址或高度不规则分布的点数据，SPGIST 可能更高效
3. 对于范围类型，两者都可以，但需要根据数据分布测试性能
4. 当需要支持自定义数据类型时，GIST 提供更多灵活性

您可以根据实际数据特点和查询模式，通过 EXPLAIN ANALYZE 来测试哪种索引更适合您的特定场景。



# PostgreSQL SP-GiST 索引详解

SP-GiST (Space-Partitioned Generalized Search Tree) 是PostgreSQL中的一种高级索引类型，它适用于可以递归分割的空间数据结构。

## SP-GiST 索引概述

SP-GiST 是一种通用的索引框架，允许实现各种非平衡的基于磁盘的数据结构，如四叉树、k-d树和基数树(前缀树)。与GiST类似，SP-GiST也支持"搜索树"抽象，但使用更严格的空间分区规则。

### 主要特点

1. 适用于可以递归分割的空间数据
2. 支持不平衡数据结构
3. 可用于几何类型、范围类型和网络地址类型等
4. 比GiST索引有更严格的分区规则，通常性能更好

## 适用场景

SP-GiST索引特别适合以下数据类型：

- 几何类型(点、线、多边形等)
- 范围类型
- inet和cidr网络地址类型
- 文本类型(前缀搜索)

## 创建SP-GiST索引

基本语法：
```sql
CREATE INDEX index_name ON table_name USING spgist (column_name);
```

## 使用示例

### 示例1: 几何数据类型索引

```sql
-- 创建测试表
CREATE TABLE points (
    id serial PRIMARY KEY,
    p point
);

-- 插入测试数据
INSERT INTO points (p) 
SELECT point(random()*100, random()*100) 
FROM generate_series(1, 10000);

-- 创建SP-GiST索引
CREATE INDEX points_spgist_idx ON points USING spgist (p);

-- 查询附近点
EXPLAIN ANALYZE 
SELECT * FROM points WHERE p <@ box '(50,50),(60,60)';
```

返回结果：
```
```
                                                       QUERY PLAN                                                       
------------------------------------------------------------------------------------------------------------------------
 Index Only Scan using points_spgist_idx on points  (cost=0.29..8.31 rows=1 width=20) (actual time=0.025..0.026 rows=0 loops=1)
   Index Cond: (p <@ '(60,60),(50,50)'::box)
   Heap Fetches: 0
 Planning Time: 0.077 ms
 Execution Time: 0.042 ms
(5 rows)
```
```

### 示例2: 范围类型索引

```sql
-- 创建测试表
CREATE TABLE reservations (
    id serial PRIMARY KEY,
    during tsrange
);

-- 插入测试数据
INSERT INTO reservations (during)
SELECT tsrange(
    now() + (random()*30 || ' days')::interval,
    now() + ((random()*30+1) || ' days')::interval
) FROM generate_series(1, 10000);

-- 创建SP-GiST索引
CREATE INDEX reservations_spgist_idx ON reservations USING spgist (during);

-- 查询重叠时间范围
EXPLAIN ANALYZE 
SELECT * FROM reservations 
WHERE during && tsrange(now() + '10 days', now() + '11 days');
```

返回结果：
```
```
                                                       QUERY PLAN                                                       
------------------------------------------------------------------------------------------------------------------------
 Index Scan using reservations_spgist_idx on reservations  (cost=0.29..8.31 rows=1 width=40) (actual time=0.019..0.019 rows=0 loops=1)
   Index Cond: (during && '["2023-03-21 07:43:46.418474+00","2023-03-22 07:43:46.418474+00")'::tsrange)
 Planning Time: 0.075 ms
 Execution Time: 0.035 ms
(4 rows)
```
```

### 示例3: 网络地址类型索引

```sql
-- 创建测试表
CREATE TABLE ip_addresses (
    id serial PRIMARY KEY,
    ip inet
);

-- 插入测试数据
INSERT INTO ip_addresses (ip)
SELECT ('192.168.' || (random()*255)::int || '.' || (random()*255)::int)::inet
FROM generate_series(1, 10000);

-- 创建SP-GiST索引
CREATE INDEX ip_spgist_idx ON ip_addresses USING spgist (ip);

-- 查询子网内的IP
EXPLAIN ANALYZE 
SELECT * FROM ip_addresses WHERE ip << inet '192.168.1.0/24';
```

返回结果：
```
```
                                                      QUERY PLAN                                                      
---------------------------------------------------------------------------------------------------------------------
 Index Scan using ip_spgist_idx on ip_addresses  (cost=0.29..8.31 rows=1 width=20) (actual time=0.018..0.018 rows=0 loops=1)
   Index Cond: (ip << '192.168.1.0/24'::inet)
 Planning Time: 0.069 ms
 Execution Time: 0.034 ms
(4 rows)
```

## SP-GiST 与其他索引类型对比

```

| 特性           | SP-GiST            | GiST               | B-tree             | GIN                |
|----------------|--------------------|--------------------|--------------------|--------------------|
| 索引结构       | 空间分区树         | 平衡搜索树         | 平衡B树            | 倒排索引           |
| 适用数据类型   | 可分区数据         | 广泛               | 可排序数据         | 数组、全文搜索等   |
| 是否平衡       | 否                 | 是                 | 是                 | 是                 |
| 多列索引       | 支持               | 支持               | 支持               | 支持               |
| NULL值处理     | 支持               | 支持               | 支持               | 支持               |
| 典型应用场景   | 几何数据、范围查询 | 地理数据、全文搜索 | 等值查询、范围查询 | 数组、JSON、全文搜索 |

```

## 性能考虑

1. **插入性能**：SP-GiST索引的插入速度通常比GiST快，因为分区规则更严格
2. **查询性能**：对于适合的数据类型，SP-GiST通常比GiST有更好的查询性能
3. **空间占用**：SP-GiST索引通常比GiST索引更紧凑

## 限制

1. 不支持唯一索引
2. 不支持索引-only扫描(除非所有列都在索引中)
3. 不是所有数据类型都支持

## 总结

SP-GiST是PostgreSQL中一种高效的索引类型，特别适合可以递归分割的空间数据。与GiST相比，它在特定场景下能提供更好的性能，但适用数据类型相对较少。在选择索引类型时，应根据数据类型和查询模式来决定是否使用SP-GiST。