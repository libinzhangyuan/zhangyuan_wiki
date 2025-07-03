[返回](/doris/knowledge/materialized-view-incremental-ori-table-requirement/index)

# 关于 Doris 中 `__DORIS_VERSION_COL__` 的建表设置

在 Apache Doris 中，**不需要也不应该**在建表时主动设置 `__DORIS_VERSION_COL__` 列。这个列是 Doris 系统自动管理的隐藏列，用户无法手动创建或修改。

## 关键说明

```
1. 自动创建：该列由Doris系统自动创建和维护
2. 隐藏属性：默认不显示在DESCRIBE结果中
3. 禁止修改：用户无法通过DDL语句操作该列
4. 全局唯一：所有表都自动拥有此列
```

## 正确建表示例

### 正常建表（无需考虑版本列）

```sql
CREATE TABLE example_db.user_behavior (
    user_id BIGINT,
    item_id BIGINT,
    behavior_type VARCHAR(20),
    ts DATETIME
)
ENGINE=OLAP
DUPLICATE KEY(user_id, item_id)
PARTITION BY RANGE(ts) (
    PARTITION p202301 VALUES LESS THAN ('2023-02-01'),
    PARTITION p202302 VALUES LESS THAN ('2023-03-01')
)
DISTRIBUTED BY HASH(user_id) BUCKETS 10;
```

### 错误示范（不要尝试）

```sql
-- 错误！不需要也不能手动添加此列
CREATE TABLE error_example (
    id INT,
    __DORIS_VERSION_COL__ BIGINT  -- 这种写法是错误的！
)
DISTRIBUTED BY HASH(id) BUCKETS 10;
```

## 如何验证版本列存在

虽然不能主动创建，但可以通过以下方式验证它的存在：

```sql
-- 1. 查询数据时显式指定
SELECT *, __DORIS_VERSION_COL__ FROM example_db.user_behavior LIMIT 1;

-- 2. 查看隐藏列（Doris 2.0+）
SHOW FULL COLUMNS FROM example_db.user_behavior;
```

可能的返回结果：
```
+-------------------+-------------+------+-------+---------+-------+
| Field             | Type        | Null | Key   | Default | Extra |
+-------------------+-------------+------+-------+---------+-------+
| user_id           | BIGINT      | Yes  | true  | NULL    |       |
| item_id           | BIGINT      | Yes  | true  | NULL    |       |
| behavior_type     | VARCHAR(20) | Yes  | false | NULL    |       |
| ts                | DATETIME    | Yes  | false | NULL    |       |
| __DORIS_VERSION_COL__ | BIGINT | No   | false | NULL    | HIDDEN |
+-------------------+-------------+------+-------+---------+-------+
```

## 版本列的实际应用场景

虽然不需要手动创建，但在以下场景可能会用到：

```sql
-- 1. 数据一致性检查（比较不同副本的版本号）
SELECT 
    tablet_id, 
    MAX(__DORIS_VERSION_COL__) as max_version 
FROM example_db.user_behavior 
GROUP BY tablet_id;

-- 2. 数据变更追踪（需配合Binlog功能）
SELECT * FROM example_db.user_behavior 
WHERE __DORIS_VERSION_COL__ > 1000;
```

## 与版本相关的合法建表参数

虽然不能直接操作版本列，但可以通过表属性间接影响版本行为：

```sql
CREATE TABLE version_controlled_table (
    id BIGINT,
    data VARCHAR(100)
)
DISTRIBUTED BY HASH(id) BUCKETS 10
PROPERTIES (
    "enable_persistent_index" = "true",  -- 持久化索引
    "storage_format_version" = "2",      -- 存储格式版本
    "disable_auto_compaction" = "false"  -- 自动压缩
);
```

## 总结建议

```
1. 完全忽略该列的建表设置
2. 需要版本查询时才显式引用该列
3. 系统自动保证该列的正确性
4. 通过表PROPERTIES调整版本相关行为
5. Doris 2.0+版本对该列的支持更完善
```