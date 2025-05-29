[返回](/doris/knowledge/index)

# Doris 动态分区功能详解

动态分区是 Apache Doris 中一个非常实用的功能，它可以自动管理分区表的生命周期，根据配置自动创建和删除分区，特别适合时间序列数据的管理。

## 动态分区基本概念

动态分区功能允许用户预先设置分区规则，系统会根据规则自动：
- 创建未来的新分区
- 删除过期的旧分区
- 保持合理数量的分区

## 动态分区配置参数
```
| 参数名 | 说明 | 示例值 |
|--------|------|--------|
| `dynamic_partition.enable` | 是否开启动态分区 | `true`/`false` |
| `dynamic_partition.time_unit` | 分区时间单位 | `DAY`/`WEEK`/`MONTH` |
| `dynamic_partition.start` | 保留分区的开始偏移量（负值） | `-3` |
| `dynamic_partition.end` | 预先创建分区的结束偏移量（正值） | `7` |
| `dynamic_partition.prefix` | 分区名前缀 | `p` |
| `dynamic_partition.buckets` | 自动创建分区的分桶数 | `8` |
| `dynamic_partition.replication_num` | 分区的副本数 | `3` |
| `dynamic_partition.start_day_of_week` | 周分区的起始日（1=周日，7=周六） | `1` |
| `dynamic_partition.start_day_of_month` | 月分区的起始日（1-28） | `1` |
```
## 动态分区配置示例

### 1. 按天自动分区

```sql
ALTER TABLE log_data SET (
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-7",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "day_",
    "dynamic_partition.buckets" = "8"
);
```
```
此配置表示：
- 保留最近7天的分区（`start = -7`）
- 预先创建未来3天的分区（`end = 3`）
- 分区名前缀为`day_`
- 每个分区8个bucket
```
### 2. 按周自动分区

```sql
ALTER TABLE weekly_reports SET (
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "WEEK",
    "dynamic_partition.start" = "-4",
    "dynamic_partition.end" = "2",
    "dynamic_partition.start_day_of_week" = "2"  -- 从周一开始
);
```

### 3. 按月自动分区

```sql
ALTER TABLE monthly_sales SET (
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "MONTH",
    "dynamic_partition.start" = "-12",
    "dynamic_partition.end" = "1",
    "dynamic_partition.prefix" = "month_"
);
```

## 查看动态分区配置

```sql
SHOW DYNAMIC PARTITION TABLES;
```

可能的返回结果：

```
+-----------+-----------+----------------+----------------+----------------+-----------+--------------+---------------------+----------------------+------------------------+---------------------+
| TableName | Enable    | TimeUnit       | Start          | End            | Prefix    | Buckets      | ReplicationNum      | StartDayOfWeek       | StartDayOfMonth        | CreateHistoryPartition |
+-----------+-----------+----------------+----------------+----------------+-----------+--------------+---------------------+----------------------+------------------------+---------------------+
| log_data  | true      | DAY            | -7             | 3              | day_      | 8            | 3                   | NULL                 | NULL                   | false              |
| monthly_sales | true  | MONTH          | -12            | 1              | month_    | 10           | 3                   | NULL                 | 1                      | true               |
+-----------+-----------+----------------+----------------+----------------+-----------+--------------+---------------------+----------------------+------------------------+---------------------+
```

## 动态分区操作示例

### 1. 创建带有动态分区配置的表

```sql
CREATE TABLE dynamic_part_table (
    dt DATE,
    id INT,
    data VARCHAR(50)
)
PARTITION BY RANGE(dt) ()
DISTRIBUTED BY HASH(id) BUCKETS 8
PROPERTIES (
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-30",
    "dynamic_partition.end" = "7",
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.buckets" = "8"
);
```

### 2. 修改现有表的动态分区配置

```sql
ALTER TABLE dynamic_part_table SET (
    "dynamic_partition.end" = "14"
);
```

### 3. 临时禁用动态分区

```sql
ALTER TABLE dynamic_part_table SET (
    "dynamic_partition.enable" = "false"
);
```

## 动态分区状态检查

```sql
-- 查看自动创建的分区
SHOW PARTITIONS FROM dynamic_part_table;

-- 查看动态分区任务执行情况
SHOW DYNAMIC PARTITION JOB FROM dynamic_part_table;
```

可能的返回结果：

```
+----------------+---------------------+------------------+-----------+--------------+-------+-----------------+---------+----------------+---------------+--------------+--------------------------+----------+------------+-------------------+-----------+
| PartitionName  | VisibleVersionTime  | VisibleVersion   | State     | PartitionKey | Range | DistributionKey | Buckets | ReplicationNum | StorageMedium | CooldownTime | LastConsistencyCheckTime | DataSize | IsInMemory | ReplicaAllocation | IsMutable |
+----------------+---------------------+------------------+-----------+--------------+-------+-----------------+---------+----------------+---------------+--------------+--------------------------+----------+------------+-------------------+-----------+
| p20230501      | 2023-05-01 00:00:00 | 1                | NORMAL    | dt           | [MIN, 2023-05-02) | id       | 8      | 3             | HDD           | 9999-12-31 23:59:59 | NULL                 | 0        | false      | {"tag.location.default":"3"} | true      |
| p20230502      | 2023-05-02 00:00:00 | 1                | NORMAL    | dt           | [2023-05-02, 2023-05-03) | id       | 8      | 3             | HDD           | 9999-12-31 23:59:59 | NULL                 | 0        | false      | {"tag.location.default":"3"} | true      |
| ...            | ...                 | ...              | ...       | ...          | ...   | ...             | ...     | ...            | ...           | ...          | ...                      | ...      | ...        | ...               | ...       |
+----------------+---------------------+------------------+-----------+--------------+-------+-----------------+---------+----------------+---------------+--------------+--------------------------+----------+------------+-------------------+-----------+
```

## 动态分区最佳实践

1. **合理设置时间范围**：
   - `start` 不宜设置过大，否则会保留过多历史分区
   - `end` 根据业务需求设置，通常预先创建1-2周的分区即可

2. **分区命名规范**：
   - 使用有意义的前缀，便于识别分区用途
   - 例如：`day_`、`week_`、`month_`等

3. **监控分区状态**：
   - 定期检查分区自动创建情况
   - 监控分区数据分布是否均匀

4. **结合冷热数据分离**：
   ```sql
   ALTER TABLE dynamic_part_table MODIFY PARTITION p202301 SET ("storage_medium" = "SSD");
   ALTER TABLE dynamic_part_table MODIFY PARTITION p202212 SET ("storage_medium" = "HDD");
   ```

动态分区功能大大简化了分区表的管理工作，特别适合处理时间序列数据，如日志、交易记录等场景。