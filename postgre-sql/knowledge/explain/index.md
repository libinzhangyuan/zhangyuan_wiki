[返回](/postgre-sql/knowledge/index)


# PostgreSQL 中 EXPLAIN 的所有选项详解

PostgreSQL 的 EXPLAIN 命令提供了多种选项来定制执行计划的输出内容。以下是所有可用选项的全面介绍：

## 基础选项

### 1. ANALYZE
**实际执行查询**并返回真实运行时统计信息
```sql
EXPLAIN ANALYZE SELECT * FROM users;
```
- 会真正执行查询
- 显示实际执行时间、返回行数等
- 生产环境使用时需谨慎

### 2. VERBOSE
**显示更详细**的信息
```sql
EXPLAIN VERBOSE SELECT * FROM users;
```
- 显示输出列列表
- 显示表的结构信息
- 适用于复杂查询调试

### 3. COSTS
**显示成本估算**（默认开启）
```sql
EXPLAIN (COSTS ON) SELECT * FROM users;  -- 默认
EXPLAIN (COSTS OFF) SELECT * FROM users; -- 不显示成本
```
- 成本包括启动成本和总成本
- 关闭后输出更简洁

### 4. TIMING
**显示时间信息**（与ANALYZE一起使用时）
```sql
EXPLAIN (ANALYZE, TIMING ON) SELECT * FROM users;  -- 默认
EXPLAIN (ANALYZE, TIMING OFF) SELECT * FROM users; -- 不显示时间细分
```
- 只影响ANALYZE的输出
- 关闭后仍显示总时间

### 5. SUMMARY
**显示总结信息**（默认开启）
```sql
EXPLAIN (SUMMARY ON) SELECT * FROM users;  -- 默认
EXPLAIN (SUMMARY OFF) SELECT * FROM users; -- 不显示总结
```
- 控制是否显示计划时间和执行时间总结
- 关闭后输出更简洁

### 6. FORMAT
**设置输出格式**
```sql
EXPLAIN (FORMAT TEXT) SELECT * FROM users;  -- 默认文本格式
EXPLAIN (FORMAT JSON) SELECT * FROM users; -- JSON格式
EXPLAIN (FORMAT XML) SELECT * FROM users;  -- XML格式
EXPLAIN (FORMAT YAML) SELECT * FROM users; -- YAML格式
```
- 支持多种机器可读格式
- 便于程序解析

## 高级选项

### 7. BUFFERS
**显示缓冲区使用情况**（需与ANALYZE一起使用）
```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM large_table;
```
- 显示共享块、本地块和临时块的读写情况
- 帮助分析I/O密集型查询

### 8. WAL
**显示WAL日志使用情况**（PostgreSQL 13+）
```sql
EXPLAIN (ANALYZE, WAL) INSERT INTO logs VALUES (...);
```
- 显示生成的WAL记录数和字节数
- 仅适用于修改数据的查询

### 9. SETTINGS
**显示非默认优化器设置**
```sql
EXPLAIN (SETTINGS ON) SELECT * FROM users;
```
- 显示影响查询计划的非默认配置参数
- 帮助理解优化器行为

## 特殊用途选项

### 10. WAL
**仅用于DML语句**（INSERT/UPDATE/DELETE）
```sql
EXPLAIN (ANALYZE, WAL) UPDATE users SET status = 'active';
```

### 11. GENERIC_PLAN
**禁止参数化计划**（PostgreSQL 12+）
```sql
EXPLAIN (GENERIC_PLAN) SELECT * FROM users WHERE id = $1;
```
- 显示不考虑特定参数值的通用计划
- 用于分析参数化查询

## 选项组合示例

```sql
-- 完整性能分析
EXPLAIN (ANALYZE, VERBOSE, BUFFERS, WAL, SUMMARY ON)
INSERT INTO audit_log SELECT * FROM transactions WHERE amount > 1000;

-- 简洁的机器可读格式
EXPLAIN (FORMAT JSON, COSTS OFF, SUMMARY OFF)
SELECT * FROM products WHERE category = 'electronics';
```

## 选项参考表
```
| 选项 | 描述 | 需要ANALYZE | 默认值 |
|------|------|------------|-------|
| ANALYZE | 实际执行查询 | 不适用 | OFF |
| VERBOSE | 显示详细信息 | 否 | OFF |
| COSTS | 显示成本估算 | 否 | ON |
| TIMING | 显示时间细分 | 是 | ON |
| SUMMARY | 显示总结信息 | 否 | ON |
| FORMAT | 输出格式(text/json/xml/yaml) | 否 | TEXT |
| BUFFERS | 显示缓冲区使用 | 是 | OFF |
| WAL | 显示WAL使用情况 | 是 | OFF |
| SETTINGS | 显示非默认设置 | 否 | OFF |
| GENERIC_PLAN | 生成通用计划 | 否 | OFF |
```
通过合理组合这些选项，您可以获取最适合当前调试需求的执行计划信息。