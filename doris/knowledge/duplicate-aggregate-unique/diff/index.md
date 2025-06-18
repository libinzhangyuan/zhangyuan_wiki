[返回](/doris/knowledge/duplicate-aggregate-unique/index)

# Doris 数据库介绍

Doris 是一个基于 MPP 架构的高性能、实时的分析型数据库，主要用于解决大规模数据的实时分析问题。下面我将详细介绍 Doris 中的三种数据模型：Duplicate、Aggregate 和 Unique。

## 1. Duplicate 模型

Duplicate 模型是 Doris 中最基础的数据模型，它不会对导入的数据进行任何处理，保留原始导入的完整数据。

特点：
- 适合存储明细数据
- 没有主键概念
- 数据完全按照导入文件中的内容存储

### 示例

```sql
-- 创建 Duplicate 表
CREATE TABLE IF NOT EXISTS example_db.duplicate_tbl (
    `timestamp` DATETIME NOT NULL COMMENT "日志时间",
    `type` INT NOT NULL COMMENT "日志类型",
    `error_code` INT COMMENT "错误码",
    `error_msg` VARCHAR(1024) COMMENT "错误详细信息",
    `op_id` BIGINT COMMENT "负责人id",
    `op_time` DATETIME COMMENT "处理时间"
)
DUPLICATE KEY(`timestamp`, `type`)
DISTRIBUTED BY HASH(`timestamp`) BUCKETS 10;
```

插入数据后查询结果：

```
+---------------------+------+------------+----------------+--------+---------------------+
| timestamp           | type | error_code | error_msg      | op_id  | op_time             |
+---------------------+------+------------+----------------+--------+---------------------+
| 2023-01-01 08:00:00 | 1    | 404        | Page not found | 1001   | 2023-01-01 08:05:00 |
| 2023-01-01 08:00:00 | 1    | 404        | Page not found | 1001   | 2023-01-01 08:05:00 |
| 2023-01-01 08:01:00 | 2    | 500        | Server error   | NULL   | NULL                |
+---------------------+------+------------+----------------+--------+---------------------+
```

## 2. Aggregate 模型

Aggregate 模型适合有聚合需求的场景，通过预聚合可以极大地提升查询效率。

特点：
- 适合有聚合需求的场景
- 数据会按照 Key 列进行聚合
- 通过预聚合提高查询效率

### 示例

```sql
-- 创建 Aggregate 表
CREATE TABLE IF NOT EXISTS example_db.aggregate_tbl (
    `user_id` LARGEINT NOT NULL COMMENT "用户id",
    `date` DATE NOT NULL COMMENT "数据灌入日期时间",
    `city` VARCHAR(20) COMMENT "用户所在城市",
    `age` SMALLINT COMMENT "用户年龄",
    `sex` TINYINT COMMENT "用户性别",
    `last_visit_date` DATETIME REPLACE DEFAULT "1970-01-01 00:00:00" COMMENT "用户最后一次访问时间",
    `cost` BIGINT SUM DEFAULT "0" COMMENT "用户总消费",
    `max_dwell_time` INT MAX DEFAULT "0" COMMENT "用户最大停留时间",
    `min_dwell_time` INT MIN DEFAULT "99999" COMMENT "用户最小停留时间"
)
AGGREGATE KEY(`user_id`, `date`, `city`, `age`, `sex`)
DISTRIBUTED BY HASH(`user_id`) BUCKETS 10;
```

插入数据后查询结果：

```
+---------+------------+-----------+------+------+---------------------+------+----------------+----------------+
| user_id | date       | city      | age  | sex  | last_visit_date     | cost | max_dwell_time | min_dwell_time |
+---------+------------+-----------+------+------+---------------------+------+----------------+----------------+
| 10001   | 2023-01-01 | 北京      | 25   | 1    | 2023-01-01 08:30:00 | 120  | 300            | 10             |
| 10002   | 2023-01-01 | 上海      | 30   | 0    | 2023-01-01 09:15:00 | 200  | 450            | 50             |
+---------+------------+-----------+------+------+---------------------+------+----------------+----------------+
```

## 3. Unique 模型

Unique 模型针对需要唯一主键的场景，可以保证主键唯一性，支持按主键更新。

特点：
- 适合有唯一性约束的场景
- 支持按主键更新
- 1.2版本后支持Merge-on-Write实现

### 示例

```sql
-- 创建 Unique 表
CREATE TABLE IF NOT EXISTS example_db.unique_tbl (
    `user_id` LARGEINT NOT NULL COMMENT "用户id",
    `username` VARCHAR(50) NOT NULL COMMENT "用户昵称",
    `city` VARCHAR(20) COMMENT "用户所在城市",
    `age` SMALLINT COMMENT "用户年龄",
    `sex` TINYINT COMMENT "用户性别",
    `phone` LARGEINT COMMENT "用户电话",
    `address` VARCHAR(500) COMMENT "用户地址",
    `register_time` DATETIME COMMENT "用户注册时间"
)
UNIQUE KEY(`user_id`)
DISTRIBUTED BY HASH(`user_id`) BUCKETS 10;
```

插入数据后查询结果：

```
+---------+-----------+-----------+------+------+-------------+---------------------+---------------------+
| user_id | username  | city      | age  | sex  | phone       | address             | register_time       |
+---------+-----------+-----------+------+------+-------------+---------------------+---------------------+
| 10001   | 张三      | 北京      | 25   | 1    | 13800138000 | 北京市海淀区        | 2022-01-01 10:00:00 |
| 10002   | 李四      | 上海      | 30   | 0    | 13900139000 | 上海市浦东新区      | 2022-01-02 11:00:00 |
+---------+-----------+-----------+------+------+-------------+---------------------+---------------------+
```

## 三种模型对比

```
+-----------------+---------------------+---------------------+---------------------+
| 特性            | Duplicate 模型      | Aggregate 模型      | Unique 模型         |
+-----------------+---------------------+---------------------+---------------------+
| 适用场景        | 存储原始明细数据    | 需要预聚合的统计    | 需要唯一主键的场景  |
| 数据去重        | 不去重              | 按Key列聚合         | 按主键去重          |
| 存储效率        | 低                  | 高                  | 中                  |
| 查询效率        | 低                  | 高                  | 中                  |
| 更新支持        | 不支持              | 部分支持            | 支持                |
| 典型应用        | 日志分析            | 报表统计            | 用户画像            |
+-----------------+---------------------+---------------------+---------------------+
```