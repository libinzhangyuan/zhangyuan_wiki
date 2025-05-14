[返回](/postgre-sql/knowledge/index)


# PostgreSQL 中的哈希索引 (Hash Index)

哈希索引是 PostgreSQL 提供的一种特殊索引类型，它使用哈希表数据结构来加速等值查询（= 操作）。

## 哈希索引的特点

1. **仅支持等值查询**：只能加速 `=` 操作，不支持范围查询（`>`, `<`, `BETWEEN` 等）
2. **无排序功能**：不保留数据的排序顺序
3. **内存效率高**：哈希表在内存中查找效率很高
4. **无事务安全**：在 PostgreSQL 10 之前，哈希索引不记录 WAL 日志，无法用于流复制
5. **大小固定**：哈希表大小在创建时确定，不会自动调整

## 创建哈希索引

```sql
CREATE INDEX index_name ON table_name USING HASH (column_name);
```

示例：
```sql
CREATE INDEX idx_user_email_hash ON users USING HASH (email);
```

## 适用场景

1. **频繁的等值查询**：当查询条件几乎总是 `column = value` 时
2. **不关心排序**：当查询不需要结果按索引列排序时
3. **高基数列**：列中不同值很多时效果更好
4. **内存充足**：哈希索引在内存中表现最佳

## 不适用场景

1. **需要范围查询**：如 `>`, `<`, `BETWEEN` 等操作
2. **需要排序结果**：如 `ORDER BY` 子句
3. **低基数列**：列中不同值很少时效果差
4. **PostgreSQL 10 之前的版本**：因为缺乏 WAL 日志支持

## PostgreSQL 10+ 的改进

从 PostgreSQL 10 开始，哈希索引有了显著改进：
1. **支持 WAL 日志**：可以用于流复制和崩溃恢复
2. **性能提升**：实现了更高效的并发控制
3. **更可靠**：适合生产环境使用

## 哈希索引 vs B-tree 索引
```
| 特性        | 哈希索引               | B-tree 索引              |
|------------|-----------------------|-------------------------|
| 查询类型    | 仅等值查询 (=)        | 等值、范围查询、排序     |
| 排序        | 不支持                | 支持                    |
| 大小        | 固定                  | 动态增长                |
| 内存使用    | 通常更高效            | 取决于数据分布          |
| 并发控制    | PostgreSQL 10+ 才完善 | 一直很完善              |
```
## 实际使用建议

1. 在 PostgreSQL 10+ 中，对于纯等值查询，哈希索引可能比 B-tree 快
2. 测试特定工作负载，因为性能差异取决于数据特征
3. 大多数情况下，B-tree 更通用，是默认选择
4. 考虑使用 `EXPLAIN ANALYZE` 比较查询计划

示例测试：
```sql
-- 创建测试索引
CREATE INDEX idx_btree ON users (email);
CREATE INDEX idx_hash ON users USING HASH (email);

-- 分析查询计划
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';
```

哈希索引是 PostgreSQL 工具箱中的一个专门工具，在特定场景下可以提供最佳性能，但需要根据具体使用情况谨慎选择。





# PostgreSQL 哈希索引的固定大小特性

在 PostgreSQL 中，哈希索引的一个关键特性是它在创建时确定大小且不会自动调整，这与 B-tree 等动态调整的索引类型不同。

## 哈希索引的固定大小机制

1. **初始化大小**：
   - 哈希索引在创建时会根据表的大小估算一个初始桶(bucket)数量
   - 这个大小在索引生命周期内保持不变

2. **工作原理**：
   - 每个桶包含指向表行的指针
   - 哈希函数将键值映射到特定桶
   - 冲突通过链表解决（同一桶中的多个条目）

3. **固定大小的影响**：
   - 不会随数据增长而自动扩展
   - 不会因数据减少而收缩

## 性能考虑

1. **过小哈希表的问题**：
   - 桶数太少会导致长冲突链
   - 查找性能退化为接近线性搜索
   - 示例：100万行数据只用100个桶

2. **过大哈希表的问题**：
   - 浪费内存和存储空间
   - 初始化时间更长
   - 示例：100行数据用10万个桶

3. **理想情况**：
   - 桶数应略大于预期唯一键值数量
   - 每个桶平均有少量条目(1-2个)

## PostgreSQL 10+ 的实现细节

从 PostgreSQL 10 开始，哈希索引实现有所改进但仍保持固定大小：

1. **桶数计算**：
   - 基于 `maintenance_work_mem` 设置
   - 使用公式：`bucket_count = next_power_of_2(estimated_count / fillfactor)`
   - 默认 fillfactor 为 75%

2. **无法调整的限制**：
   ```sql
   -- 创建后无法更改大小
   CREATE INDEX idx_hash ON table USING HASH (column); -- 大小已固定
   ```

3. **重建索引**：
   - 唯一调整方法是删除并重建索引
   - 重建时会根据当前数据量重新计算大小

## 与动态索引的对比
```
| 特性            | 哈希索引                     | B-tree 索引               |
|----------------|-----------------------------|--------------------------|
| 大小调整        | 固定，创建时确定            | 动态增长和收缩            |
| 数据增长影响    | 性能可能下降                | 自动保持平衡              |
| 数据减少影响    | 空间不回收                  | 空间可回收                |
| 维护需求        | 可能需要定期重建            | 通常自动维护              |
```
## 最佳实践建议

1. **预估数据量**：
   - 在创建哈希索引前预估列的唯一值数量
   - 对大表考虑先采样分析

2. **监控性能**：
   ```sql
   -- 检查索引膨胀
   SELECT * FROM pgstatindex('idx_hash');
   
   -- 查看索引统计
   SELECT * FROM pg_stats WHERE tablename = 'your_table';
   ```

3. **重建策略**：
   - 定期监控并重建性能下降的哈希索引
   - 在数据量变化显著后考虑重建

4. **替代方案**：
   - 如果数据量变化大，考虑使用 B-tree
   - 对已知固定规模的数据集使用哈希索引

这种固定大小的特性使得哈希索引特别适合数据量稳定且已知的场合，而对于增长不可预测的数据集则需要谨慎使用。



# PostgreSQL 哈希索引的桶大小设置

在 PostgreSQL 中，**不能直接指定哈希索引的桶(bucket)数量或大小**，这是哈希索引实现的一个限制。不过有一些间接方法可以影响哈希索引的初始大小。

## 当前实现限制

1. **无显式参数**：
   PostgreSQL 没有提供类似 `WITH (buckets=1000)` 这样的语法来直接设置桶数量。

2. **自动计算**：
   桶数量由系统根据以下因素自动计算：
   - 表的大小估计值
   - `maintenance_work_mem` 参数设置
   - 填充因子(fillfactor)

## 间接影响桶数量的方法

1. **通过 maintenance_work_mem 调整**：
   ```sql
   -- 在创建索引前临时增大(会话级)
   SET maintenance_work_mem = '256MB';
   CREATE INDEX idx_hash ON table USING HASH (column);
   ```

2. **填充因子(fillfactor)**：
   ```sql
   -- 减少填充因子会使系统分配更多桶
   CREATE INDEX idx_hash ON table USING HASH (column) WITH (fillfactor=50);
   ```
   - 默认 fillfactor 为 75%
   - 较低的值 = 更多桶 + 每个桶更少条目

3. **在加载数据前创建索引**：
   - 空表上创建的哈希索引会很小
   - 建议先加载数据再创建哈希索引

## 查看实际桶数量

创建后可以检查实际分配的桶数：
```sql
SELECT pg_relpages('idx_hash')*current_setting('block_size')::int/1024 || 'KB' AS index_size;
```

或使用扩展：
```sql
CREATE EXTENSION pgstattuple;
SELECT * FROM pgstatindex('idx_hash');
```

## 替代方案建议

如果需要精确控制索引结构：

1. **考虑 B-tree 索引**：
   - 完全动态调整
   - 支持更多查询类型

2. **手动分桶设计**：
   ```sql
   -- 例如按哈希值手动分片
   CREATE TABLE partitioned_table (
     id serial,
     data text,
     bucket int GENERATED ALWAYS AS (abs(hashint4(id)) % 100) STORED
   ) PARTITION BY LIST (bucket);
   ```

3. **等待未来版本**：
   PostgreSQL 社区仍在改进哈希索引，未来版本可能会提供更多控制参数。

虽然无法直接设置桶大小，但通过合理配置 `maintenance_work_mem` 和 `fillfactor`，可以在大多数情况下获得性能良好的哈希索引。