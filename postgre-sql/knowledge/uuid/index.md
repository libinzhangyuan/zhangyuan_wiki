[返回](/postgre-sql/knowledge/index)

# 提示，请用uuidv7 - 张远



# PostgreSQL 中的 UUID 类型

UUID (Universally Unique Identifier) 是 PostgreSQL 中用于存储全局唯一标识符的数据类型。UUID 是一个 128 位的值，通常表示为 32 个十六进制数字，由连字符分隔为五组，形式如 `123e4567-e89b-12d3-a456-426614174000`。

## UUID 的特点

1. **全局唯一性**：理论上在分布式系统中生成重复 UUID 的概率极低
2. **无需中央权威机构**：可以离线生成
3. **无序性**：UUID 不是按顺序生成的，不适合作为聚集索引
4. **固定大小**：总是 16 字节 (128 位)

## PostgreSQL 中的 UUID 使用

### 启用 UUID 支持

要使用 UUID 类型，需要先启用 `uuid-ossp` 扩展：

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

### 生成 UUID

PostgreSQL 提供了几种生成 UUID 的方法：

1. 使用 `uuid-ossp` 扩展的函数：
   ```sql
   SELECT uuid_generate_v1();  -- 基于时间戳和 MAC 地址
   SELECT uuid_generate_v1mc(); -- 类似 v1，但使用随机多播 MAC
   SELECT uuid_generate_v3(namespace uuid, name text); -- 基于 MD5 哈希
   SELECT uuid_generate_v4();  -- 完全随机
   SELECT uuid_generate_v5(namespace uuid, name text); -- 基于 SHA1 哈希
   ```

2. 从 PostgreSQL 13 开始，可以使用内置函数：
   ```sql
   SELECT gen_random_uuid();  -- 等同于 uuid_generate_v4()
   ```

### 创建包含 UUID 列的表

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL
);
```

### 插入数据

```sql
-- 使用默认值自动生成 UUID
INSERT INTO users (username, email) VALUES ('john_doe', 'john@example.com');

-- 显式指定 UUID
INSERT INTO users (id, username, email) 
VALUES ('a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11', 'jane_doe', 'jane@example.com');
```

## UUID 版本比较
```
| 版本 | 描述 | 特点 |
|------|------|------|
| v1   | 基于时间戳和 MAC 地址 | 包含生成时间和机器标识，可能泄露隐私 |
| v3   | 基于命名空间和名称的 MD5 哈希 | 确定性生成，相同输入产生相同输出 |
| v4   | 随机生成 | 最常用，完全随机 |
| v5   | 基于命名空间和名称的 SHA1 哈希 | 类似 v3，但使用更安全的哈希算法 |
| v6   | 时间排序 | v1 的重新排序版本，更适合作为数据库主键 | 需要 `uuidv6` 扩展 |
| v7   | 时间排序 | 基于 Unix 时间戳 + 随机数，更适合现代应用 | PostgreSQL 16+ 原生支持 | (推荐都用这个)
| v8   | 自定义 | 自由格式，保留 UUID 结构但允许自定义内容 | 需要第三方扩展 |





```
## 使用建议

1. **主键选择**：UUID 适合作为分布式系统的主键，但比自增整数占用更多空间
2. **索引性能**：由于 UUID 的无序性，可能导致索引碎片化
3. **存储空间**：UUID 占用 16 字节，比 BIGINT (8 字节) 大
4. **显示格式**：PostgreSQL 会自动将 UUID 格式化为标准字符串表示

UUID 在需要全局唯一标识符或分布式数据库环境中特别有用，但在简单的单机应用中，传统的序列可能仍然是更好的选择。










# PostgreSQL 中的 UUID v7 详解

UUID v7 是最新提出的 UUID 版本，专门为解决传统 UUID 在数据库系统中的一些痛点而设计。它结合了时间排序性和随机性，非常适合作为数据库主键。

## UUID v7 的核心特点

1. **时间排序性**：
   - 前 48 位存储 Unix 时间戳（毫秒精度）
   - 生成的 UUID 会按时间顺序递增
   - 解决了 v4 UUID 完全随机导致的索引碎片问题

2. **高随机性**：
   - 后 74 位使用加密安全的随机数
   - 保证了足够的唯一性

3. **隐私安全**：
   - 不包含 MAC 地址等硬件信息（与 v1 不同）
   - 不会泄露生成设备的信息

4. **标准化**：
   - 已被纳入最新的 UUID 标准草案
   - PostgreSQL 16+ 原生支持

## UUID v7 的二进制结构

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           unix_ts_ms                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          unix_ts_ms           |  ver  |       rand_a          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|var|                        rand_b                             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                            rand_b                             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- **unix_ts_ms** (48位): Unix 时间戳，毫秒精度
- **ver** (4位): 版本号 (0111)
- **rand_a** (12位): 随机数
- **var** (2位): 变体号 (10)
- **rand_b** (62位): 更多随机数

## PostgreSQL 中的使用

### PostgreSQL 16+ 原生支持

```sql
-- 生成 v7 UUID
SELECT gen_random_uuid_v7();

-- 创建表使用 v7 作为主键
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid_v7(),
    order_data JSONB NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 旧版本 PostgreSQL 使用扩展

```sql
-- 安装 uuidv7 扩展
CREATE EXTENSION IF NOT EXISTS "uuidv7";

-- 生成 v7 UUID
SELECT uuid_generate_v7();
```

## UUID v7 的优势场景

1. **数据库主键**：
   - 时间排序性减少索引碎片
   - 比自增整数更适合分布式系统

2. **日志记录**：
   - 可以按 ID 排序即按时间排序
   - 比纯时间戳更安全

3. **分布式系统**：
   - 无需协调即可生成
   - 各节点生成的 ID 仍保持大致时间顺序

4. **时间序列数据**：
   - 既有时序性又有唯一性
   - 比组合 (时间戳+随机数) 更规范

## 性能考虑

1. **生成速度**：
   - 比 v4 稍慢（需要获取当前时间）
   - 比 v1 快（不需要获取 MAC 地址）

2. **存储效率**：
   - 与所有 UUID 版本相同（16 字节）
   - 比 BIGINT (8 字节) 大，但比组合列小

3. **索引效率**：
   - 远好于 v4（减少随机插入导致的碎片）
   - 略逊于纯自增整数

## 与其它版本的比较
```
| 特性        | v4          | v7          | v1          |
|------------|-------------|-------------|-------------|
| 有序性      | 完全随机    | 时间排序    | 时间排序    |
| 隐私安全    | 是          | 是          | 否 (含 MAC) |
| 分布式友好  | 是          | 是          | 部分        |
| 索引碎片    | 高          | 低          | 低          |
| 时间精度    | 无          | 毫秒        | 100ns       |
```
## 实际应用示例

```sql
-- 创建分布式订单表
CREATE TABLE distributed_orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid_v7(),
    user_id UUID NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    status VARCHAR(20) NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 批量插入
INSERT INTO distributed_orders (user_id, amount, status)
SELECT 
    gen_random_uuid(), 
    (random() * 1000)::DECIMAL(10,2),
    CASE WHEN random() > 0.5 THEN 'completed' ELSE 'pending' END
FROM generate_series(1, 1000);

-- 查询会自动按时间顺序排列
SELECT * FROM distributed_orders ORDER BY id LIMIT 10;
```

UUID v7 是现代应用中最推荐的 UUID 版本，特别是在 PostgreSQL 16+ 环境中。它平衡了唯一性、有序性和隐私性，是传统自增ID和随机UUID的优秀替代方案。