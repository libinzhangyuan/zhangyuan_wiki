[返回](/doris/knowledge/index)

# Doris 数据库中的 Partition（分区）介绍

Partition 是 Apache Doris 中非常重要的数据组织方式，它可以帮助提高查询效率、方便数据管理，并优化数据生命周期管理。

## 基本概念
```
Partition（分区）是指按照一定的规则将表数据划分为不同的部分，每个分区可以独立管理。Doris 支持两种分区方式：
1. **Range Partition**：按照范围分区（如按日期、数值范围）
2. **List Partition**：按照枚举值分区（如按地区、类别）
```
## 分区的作用

1. **查询优化**：查询时可以只扫描相关分区，提高查询效率
2. **数据管理**：可以单独对某个分区进行删除、备份等操作
3. **生命周期管理**：可以设置分区的存储策略和生命周期

## 分区创建示例

### 1. 按日期范围分区

```sql
CREATE TABLE sales_records (
    sale_date DATE,
    customer_id INT,
    amount DECIMAL(10,2)
)
PARTITION BY RANGE(sale_date) (
    PARTITION p202301 VALUES LESS THAN ('2023-02-01'),
    PARTITION p202302 VALUES LESS THAN ('2023-03-01'),
    PARTITION p202303 VALUES LESS THAN ('2023-04-01'),
    PARTITION p202304 VALUES LESS THAN ('2023-05-01')
)
DISTRIBUTED BY HASH(customer_id) BUCKETS 10;
```

### 2. 按枚举值分区

```sql
CREATE TABLE user_profiles (
    user_id BIGINT,
    region VARCHAR(50),
    gender VARCHAR(10),
    age INT
)
PARTITION BY LIST(region) (
    PARTITION p_east VALUES IN ("Beijing", "Shanghai", "Tianjin"),
    PARTITION p_south VALUES IN ("Guangzhou", "Shenzhen", "Hangzhou"),
    PARTITION p_west VALUES IN ("Chengdu", "Chongqing", "Xian")
)
DISTRIBUTED BY HASH(user_id) BUCKETS 8;
```

## 分区管理操作

### 查看表的分区信息

```sql
SHOW PARTITIONS FROM sales_records;
```

可能的返回结果：

```
+-------------+----------------+------------------+-----------+--------------+-------------------+----------------+---------+----------------+---------------+---------------------+---------------------+--------------------------+----------+------------+-------------------------+-----------+
| PartitionId | PartitionName  | VisibleVersion   | VisibleVersionTime | State | PartitionKey | Range | DistributionKey | Buckets | ReplicationNum | StorageMedium | CooldownTime | LastConsistencyCheckTime | DataSize | IsInMemory | ReplicaAllocation | IsMutable |
+-------------+----------------+------------------+---------------------+-------+--------------+-------+-----------------+---------+----------------+---------------+--------------+--------------------------+----------+------------+-------------------+-----------+
| 10001       | p202301        | 3                | 2023-01-15 10:00:00 | NORMAL | sale_date    | [MIN, 2023-02-01) | customer_id | 10     | 3             | HDD           | 9999-12-31 23:59:59 | NULL                 | 2.3 GB   | false      | {"tag.location.default":"3"} | true      |
| 10002       | p202302        | 2                | 2023-02-10 09:30:00 | NORMAL | sale_date    | [2023-02-01, 2023-03-01) | customer_id | 10     | 3             | HDD           | 9999-12-31 23:59:59 | NULL                 | 1.8 GB   | false      | {"tag.location.default":"3"} | true      |
| 10003       | p202303        | 1                | 2023-03-05 14:15:00 | NORMAL | sale_date    | [2023-03-01, 2023-04-01) | customer_id | 10     | 3             | HDD           | 9999-12-31 23:59:59 | NULL                 | 2.1 GB   | false      | {"tag.location.default":"3"} | true      |
+-------------+----------------+------------------+---------------------+-------+--------------+-------+-----------------+---------+----------------+---------------+--------------+--------------------------+----------+------------+-------------------+-----------+
```

### 添加新分区

```sql
ALTER TABLE sales_records ADD PARTITION p202305 VALUES LESS THAN ('2023-06-01');
```

### 删除分区

```sql
ALTER TABLE sales_records DROP PARTITION p202301;
```

### 查看分区数据量

```sql
SELECT PARTITION_NAME, TABLE_ROWS 
FROM INFORMATION_SCHEMA.PARTITIONS 
WHERE TABLE_NAME = 'sales_records';
```

可能的返回结果：

```
+----------------+------------+
| PARTITION_NAME | TABLE_ROWS |
+----------------+------------+
| p202302        | 125000     |
| p202303        | 142000     |
| p202304        | 98000      |
+----------------+------------+
```

## 分区剪枝（Partition Pruning）

Doris 会自动进行分区剪枝，只扫描查询相关的分区。例如：

```sql
-- 只扫描p202302分区
SELECT * FROM sales_records WHERE sale_date BETWEEN '2023-02-01' AND '2023-02-28';
```

## 动态分区

Doris 支持动态分区功能，可以自动创建和删除分区：

```sql
-- 启用动态分区
ALTER TABLE sales_records SET (
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "MONTH",
    "dynamic_partition.start" = "-3",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.buckets" = "10"
);
```

## 最佳实践

1. 选择合适的分区键，通常是查询条件中常用的列
2. 分区数量不宜过多，一般建议不超过1000个
3. 对于时间序列数据，建议按时间分区
4. 结合业务场景设计分区策略，如按地区、业务线等

通过合理使用分区，可以显著提高 Doris 的查询性能和管理效率。