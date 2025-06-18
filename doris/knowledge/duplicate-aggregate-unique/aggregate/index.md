[返回](/doris/knowledge/duplicate-aggregate-unique/index)

# Doris 数据库 Aggregate 模型介绍

Apache Doris 的 Aggregate 模型是一种专门为聚合查询优化的数据模型，它通过在数据导入时预先聚合数据来显著提高查询性能。

## 基本概念

Aggregate 模型的核心思想是"预先聚合"，即在数据导入阶段就对需要聚合的列进行聚合计算，这样在查询时可以直接使用预计算的结果，而不需要实时计算。

## 适用场景

Aggregate 模型非常适合以下场景：
- 需要频繁执行 SUM、COUNT、MAX、MIN 等聚合操作的业务
- 数据量大但需要快速响应的聚合查询
- 报表类应用

## 创建表示例

```sql
CREATE TABLE sales_records (
    user_id LARGEINT NOT NULL COMMENT "用户id",
    date DATE NOT NULL COMMENT "数据灌入日期时间",
    city VARCHAR(20) COMMENT "用户所在城市",
    age SMALLINT COMMENT "用户年龄",
    last_visit_date DATETIME REPLACE DEFAULT "1970-01-01 00:00:00" COMMENT "用户最后一次访问时间",
    total_cost BIGINT SUM DEFAULT "0" COMMENT "用户总消费",
    max_dwell_time INT MAX DEFAULT "0" COMMENT "用户最大停留时间",
    min_dwell_time INT MIN DEFAULT "99999" COMMENT "用户最小停留时间"
)
AGGREGATE KEY(user_id, date, city, age)
DISTRIBUTED BY HASH(user_id) BUCKETS 10
PROPERTIES (
    "replication_num" = "3"
);
```

## 聚合函数类型

Doris Aggregate 模型支持以下聚合类型：

```
1. SUM：求和
2. MIN：求最小值
3. MAX：求最大值
4. REPLACE：替换(保留最后一次导入的值)
5. REPLACE_IF_NOT_NULL：非空值替换
6. HLL_UNION：HLL 类型列的聚合
7. BITMAP_UNION：Bitmap 类型列的聚合
```

## 数据导入与查询示例

### 导入数据

```sql
INSERT INTO sales_records VALUES
(10000, '2023-05-01', '北京', 20, '2023-05-01 08:00:00', 100, 30, 10),
(10000, '2023-05-01', '北京', 20, '2023-05-01 09:00:00', 50, 20, 5),
(10001, '2023-05-01', '上海', 25, '2023-05-01 10:00:00', 200, 45, 15);
```

### 查询数据

```sql
SELECT * FROM sales_records;
```

查询结果：

```
+---------+------------+--------+------+---------------------+------------+----------------+----------------+
| user_id | date       | city   | age  | last_visit_date     | total_cost | max_dwell_time | min_dwell_time |
+---------+------------+--------+------+---------------------+------------+----------------+----------------+
| 10000   | 2023-05-01 | 北京   | 20   | 2023-05-01 09:00:00 | 150        | 30             | 5              |
| 10001   | 2023-05-01 | 上海   | 25   | 2023-05-01 10:00:00 | 200        | 45             | 15             |
+---------+------------+--------+------+---------------------+------------+----------------+----------------+
```

可以看到 user_id=10000 的两条记录已经按照 Aggregate 模型的规则进行了聚合：
- last_visit_date 使用了 REPLACE 保留了最后的值
- total_cost 使用了 SUM 进行了求和
- max_dwell_time 使用了 MAX 取了最大值
- min_dwell_time 使用了 MIN 取了最小值

## 与其他模型的对比

```
| 特性                | Aggregate 模型       | Unique 模型          | Duplicate 模型      |
|---------------------|----------------------|----------------------|----------------------|
| 是否预聚合          | 是                   | 否                   | 否                   |
| 存储空间            | 较小                 | 中等                 | 较大                 |
| 写入性能            | 中等                 | 较低                 | 较高                 |
| 聚合查询性能        | 非常高               | 中等                 | 较低                 |
| 适用场景            | 聚合分析             | 主键唯一性要求       | 原始数据存储         |
| 是否支持更新        | 通过聚合函数间接支持 | 支持                 | 不支持               |
```

## 注意事项

1. Aggregate 模型的表中所有 VALUE 列都必须指定聚合函数
2. 查询时 GROUP BY 子句必须包含所有 KEY 列
3. 对于不需要聚合的列可以使用 REPLACE 函数
4. 导入数据时相同 KEY 的行会自动聚合

## 高级用法示例

### 使用 HLL 进行 UV 统计

```sql
CREATE TABLE uv_analysis (
    date DATE NOT NULL COMMENT "日期",
    page_id LARGEINT NOT NULL COMMENT "页面ID",
    user_set HLL HLL_UNION COMMENT "用户集合"
)
AGGREGATE KEY(date, page_id)
DISTRIBUTED BY HASH(date) BUCKETS 8;

-- 导入数据
INSERT INTO uv_analysis VALUES ('2023-05-01', 1001, hll_hash(10000));
INSERT INTO uv_analysis VALUES ('2023-05-01', 1001, hll_hash(10001));

-- 查询UV
SELECT date, page_id, HLL_CARDINALITY(user_set) AS uv 
FROM uv_analysis;
```

查询结果：

```
+------------+---------+------+
| date       | page_id | uv   |
+------------+---------+------+
| 2023-05-01 | 1001    | 2    |
+------------+---------+------+
```

Aggregate 模型是 Doris 中用于高效处理聚合查询的核心特性，合理使用可以大幅提升分析查询的性能。