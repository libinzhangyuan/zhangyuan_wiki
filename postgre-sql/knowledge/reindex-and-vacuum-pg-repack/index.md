[返回](/postgre-sql/knowledge/index)

# PostgreSQL 维护工具深度比较：REINDEX vs VACUUM vs pg_repack

## 1. REINDEX

**核心功能**：重建索引解决索引膨胀问题
```
**特点**：
- 完全重新创建索引文件
- 标准版本会锁表（PostgreSQL 12+ 支持 CONCURRENTLY 不锁表）
- 适用于索引膨胀严重的情况
```
**典型用法**：
```sql
-- 阻塞式重建（简单但锁表）
REINDEX INDEX idx_name;
REINDEX TABLE table_name;

-- 非阻塞式重建（PostgreSQL 12+）
REINDEX INDEX CONCURRENTLY idx_name;
```

```
**优点**：
- 彻底解决索引膨胀
- 可以修复索引损坏
- 更新索引统计信息

**缺点**：
- 标准版本阻塞读写
- 资源消耗大
- 需要临时存储空间
```
## 2. VACUUM

**核心功能**：回收死元组空间，维护数据库健康

**类型对比**：
```
| 类型 | 锁级别 | 空间回收 | 其他特性 |
|------|--------|----------|----------|
| **常规VACUUM** | 不锁表 | 部分回收 | 可并行执行 |
| **VACUUM FULL** | 排他锁 | 完全回收 | 重建表文件 |
| **AUTOVACUUM** | 自动调度 | 渐进回收 | 后台进程 |
```
**典型用法**：
```sql
-- 常规维护（不锁表）
VACUUM ANALYZE table_name;

-- 彻底整理（锁表）
VACUUM FULL table_name;
```

```
**优点**：
- 常规VACUUM不影响生产
- 自动防止事务ID回卷
- 维护成本低

**缺点**：
- VACUUM FULL阻塞严重
- 不能完全消除膨胀
- 需要定期执行
```
## 3. pg_repack

**核心功能**：完全在线重建表和索引（扩展工具）

**核心特性**：
- 无锁重组表和索引
- 不需要额外磁盘空间
- 保持在线访问

**安装使用**：
```sql
-- 安装扩展
CREATE EXTENSION pg_repack;

-- 重组表
pg_repack -d dbname -t table_name

-- 重组索引
pg_repack -d dbname --only-indexes -t table_name
```

**优点**：
- 零停机维护
- 彻底解决膨胀
- 比VACUUM FULL更安全

**缺点**：
- 需要安装扩展
- 资源消耗较高
- 执行时间较长

## 三维度对比
```
| 维度        | REINDEX               | VACUUM            | pg_repack |
|------      |--------              -|--------|-----------|
| **锁级别**  | 阻塞/CONCURRENTLY不阻塞 | 常规不阻塞/FULL阻塞 | 完全不阻塞 |
| **空间回收** | 仅索引                 | 表+索引（部分/完全） | 表+索引（完全） |
| **适用场景** | 索引膨胀               | 常规维护/严重膨胀    | 关键业务表维护 |
| **资源消耗** | 高                    | 中-高              | 非常高 |
| **执行频率** | 按需                  | 定期                | 按需 |
| **维护难度** | 简单                  | 简单                | 需安装扩展 |
```
## 生产环境最佳实践

1. **日常维护**：
   ```sql
   -- 启用并优化autovacuum
   ALTER SYSTEM SET autovacuum = on;
   ALTER SYSTEM SET autovacuum_vacuum_scale_factor = 0.05;
   
   -- 定期执行
   VACUUM ANALYZE;
   ```

2. **月度维护**：
   ```sql
   -- 非关键时段执行
   REINDEX INDEX CONCURRENTLY idx_important;
   pg_repack -d production -t large_table --no-order --wait-timeout=3600
   ```

3. **紧急情况**：
   ```sql
   -- 维护窗口内执行
   VACUUM FULL problem_table;
   REINDEX TABLE problem_table;
   ```

4. **监控方案**：
   ```sql
   -- 膨胀监控查询
   SELECT
       nspname || '.' || relname AS table,
       pg_size_pretty(pg_relation_size(relid)) AS size,
       pg_size_pretty(pg_total_relation_size(relid) - pg_relation_size(relid)) AS index_size,
       n_dead_tup
   FROM pg_catalog.pg_stat_all_tables
   WHERE n_dead_tup > 0
   ORDER BY n_dead_tup DESC LIMIT 20;
   ```

## 选择指南

1. **常规维护**：使用 AUTOVACUUM + 定期 VACUUM ANALYZE
2. **索引问题**：REINDEX CONCURRENTLY
3. **大表重组**：优先考虑 pg_repack
4. **紧急修复**：在维护窗口使用 VACUUM FULL + REINDEX

三种工具各有所长，合理搭配使用可以确保PostgreSQL数据库长期保持高性能状态。