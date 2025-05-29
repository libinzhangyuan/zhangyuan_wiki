[返回](/doris/knowledge/index)


# Doris 中的 Partition、Bucket 和 Tablet 详解

在 Apache Doris 中，数据分布是通过 Partition(分区)、Bucket(分桶) 和 Tablet(数据分片) 三级结构来组织的。下面我将详细介绍这三者的概念、关系以及使用示例。

## 1. 基本概念对比
```
| 概念      | 说明                                                                 | 类比                  | 是否物理概念 |
|-----------|----------------------------------------------------------------------|-----------------------|-------------|
| Partition | 按照分区键将表数据划分为不同的逻辑单元，常用于分区裁剪(partition pruning) | 类似文件系统的文件夹  | 逻辑概念    |
| Bucket    | 在分区内按照分布键将数据哈希到不同的分桶中                           | 类似哈希分区          | 逻辑概念    |
| Tablet    | 分桶对应的物理数据分片，是数据移动和复制的最小单元                   | 类似HDFS的block       | 物理概念    |
```
## 2. Partition (分区)

### 作用
- 数据管理：可以按分区进行数据生命周期管理
- 查询加速：分区裁剪可以减少扫描数据量
- 并行计算：不同分区可以并行处理

### 创建分区表示例

```sql
CREATE TABLE sales_records (
    record_id BIGINT,
    sale_date DATE,
    customer_id BIGINT,
    amount DECIMAL(10,2)
)
PARTITION BY RANGE(sale_date) (
    PARTITION p202301 VALUES LESS THAN ('2023-02-01'),
    PARTITION p202302 VALUES LESS THAN ('2023-03-01'),
    PARTITION p202303 VALUES LESS THAN ('2023-04-01')
)
DISTRIBUTED BY HASH(customer_id) BUCKETS 10;
```

### 查看分区信息

```sql
SHOW PARTITIONS FROM sales_records;
```

可能的返回结果：
```
+-------------+---------------+----------------+---------+--------------+---------------------+--------+--------------+---------------+-------------------+-------------------+---------+-----------+----------------+---------------------+
| PartitionId | PartitionName | VisibleVersion | State   | PartitionKey | Range               | Distribution | BucketsNum | ReplicationNum | StorageMedium     | CooldownTime      | LastConsistencyCheckTime | DataSize | IsInMemory | ReplicaAllocation | 
+-------------+---------------+----------------+---------+--------------+---------------------+--------+--------------+---------------+-------------------+-------------------+---------+-----------+----------------+---------------------+
| 10001       | p202301       | 1              | NORMAL  | sale_date    | [types: [DATE]; keys: ["0000-01-01"]; ..["2023-02-01"]) | HASH(customer_id) | 10 | 3 | HDD | NULL | NULL | 0.000 | false | tag.location.default: 3 |
| 10002       | p202302       | 1              | NORMAL  | sale_date    | [types: [DATE]; keys: ["2023-02-01"]; ..["2023-03-01"]) | HASH(customer_id) | 10 | 3 | HDD | NULL | NULL | 0.000 | false | tag.location.default: 3 |
| 10003       | p202303       | 1              | NORMAL  | sale_date    | [types: [DATE]; keys: ["2023-03-01"]; ..["2023-04-01"]) | HASH(customer_id) | 10 | 3 | HDD | NULL | NULL | 0.000 | false | tag.location.default: 3 |
+-------------+---------------+----------------+---------+--------------+---------------------+--------+--------------+---------------+-------------------+-------------------+---------+-----------+----------------+---------------------+
```

## 3. Bucket (分桶)

### 特点
- 每个分区内的数据会进一步分桶
- 分桶数一旦确定不能修改
- 相同分桶键的数据会落在同一个桶中

### 分桶数计算示例

假设：
- 总数据量：1TB
- 每个Tablet建议大小：1GB
- 副本数：3

则分桶数 = 总数据量 / (Tablet大小 × 副本数) = 1024 / (1×3) ≈ 342

实际创建时可取整为 256 或 512（2的幂次）

## 4. Tablet (数据分片)

### 关键点
- 是物理存储单元
- 默认有3个副本（可配置）
- 副本分布在不同的BE节点上

### 查看Tablet信息

```sql
-- 先获取表的Tablet分布情况
SHOW TABLETS FROM sales_records;

-- 然后查看具体Tablet信息
SHOW TABLET 10001_1_1_1;  -- TabletId格式通常为PartitionId_TabletIndex_ReplicaId_版本号
```

可能的返回结果：
```
+----------+-----------+-----------+------------+---------+-------------+------------------+---------------------+---------------------+------------+----------+---------------------+
| TabletId | ReplicaId | BackendId | SchemaHash | Version | VersionHash | LstSuccessVersion | LstSuccessVersionHash | LstFailedVersion | LstFailedTime | DataSize | RemoteDataSize | 
+----------+-----------+-----------+------------+---------+-------------+------------------+---------------------+---------------------+------------+----------+---------------------+
| 10001_1  | 10001_1_1 | 10003     | 12345678   | 5       | 987654321   | 5                | 987654321           | 0                  | NULL       | 1.2 GB   | 0.000               |
+----------+-----------+-----------+------------+---------+-------------+------------------+---------------------+---------------------+------------+----------+---------------------+
```

## 5. 三级结构关系

```
Table → Partitions → Buckets → Tablets
```

- 一个表可以有多个分区(Partition)
- 一个分区可以有多个分桶(Bucket)
- 一个分桶对应多个Tablet(由副本数决定)

**示例关系图**：

```
sales_records (表)
├── p202301 (分区)
│   ├── Bucket 0
│   │   ├── Tablet 10001_0 (副本1)
│   │   ├── Tablet 10001_0 (副本2)
│   │   └── Tablet 10001_0 (副本3)
│   ├── Bucket 1
│   │   ├── Tablet 10001_1 (副本1)
│   │   └── ...
│   └── ...
├── p202302 (分区)
│   └── ...
└── p202303 (分区)
    └── ...
```

## 6. 最佳实践

### 分区设计建议
1. 按时间分区是最常见的做法
2. 单个分区数据量建议在1GB-10GB
3. 避免创建过多小分区（不超过1万）

```sql
-- 动态分区示例
CREATE TABLE dynamic_partition_table (
    dt DATE,
    id INT,
    data VARCHAR(100)
)
PARTITION BY RANGE(dt) (
    PARTITION p202301 VALUES LESS THAN ('2023-02-01')
)
DISTRIBUTED BY HASH(id) BUCKETS 10
PROPERTIES (
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "MONTH",
    "dynamic_partition.start" = "-12",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.buckets" = "10"
);
```

### 分桶设计建议
1. 分桶数建议是BE节点数的整数倍
2. 大表分桶数可以多些(几十到几百)
3. 小表可以设置较少分桶数(5-10个)

```sql
-- 根据集群规模自动设置分桶数
SET enable_auto_bucket = true;
CREATE TABLE auto_bucket_table (
    id BIGINT,
    data VARCHAR(100)
)
PARTITION BY RANGE(id) (
    PARTITION p0 VALUES LESS THAN (1000000),
    PARTITION p1 VALUES LESS THAN (2000000)
)
DISTRIBUTED BY HASH(id);  -- 不指定BUCKETS数量
```

## 7. 常见问题处理

### 数据倾斜检查

```sql
-- 检查分区大小
SHOW PARTITIONS FROM sales_records ORDER BY DataSize DESC;

-- 检查Tablet大小差异
SELECT 
    PartitionId, 
    AVG(DataSize) as avg_size,
    MAX(DataSize) as max_size,
    MIN(DataSize) as min_size,
    MAX(DataSize)/AVG(DataSize) as skew_ratio
FROM information_schema.tablets
WHERE TableName = 'sales_records'
GROUP BY PartitionId;
```

可能的返回结果：
```
+-------------+-----------+-----------+-----------+-----------+
| PartitionId | avg_size  | max_size  | min_size  | skew_ratio|
+-------------+-----------+-----------+-----------+-----------+
| 10001       | 1.2 GB    | 3.5 GB    | 0.8 GB    | 2.92      |
| 10002       | 1.1 GB    | 1.3 GB    | 0.9 GB    | 1.18      |
+-------------+-----------+-----------+-----------+-----------+
```