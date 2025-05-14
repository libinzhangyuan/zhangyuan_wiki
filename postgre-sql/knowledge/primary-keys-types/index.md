[返回](/postgre-sql/knowledge/index)

# PostgreSQL 主键类型详解

PostgreSQL 提供了多种主键类型选择，每种类型适用于不同的场景。以下是主要的主键类型及其特性：

## 1. 序列整型 (Serial Types)

最常用的主键类型，自动递增：

### a. SMALLSERIAL
- 2字节存储
- 范围: -32,768 到 32,767
- 适用于小型表
```sql
CREATE TABLE small_table (
    id SMALLSERIAL PRIMARY KEY,
    name VARCHAR(100)
);
```

### b. SERIAL
- 4字节存储 (最常用)
- 范围: -2,147,483,648 到 2,147,483,647
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE
);
```

### c. BIGSERIAL
- 8字节存储
- 范围: -9,223,372,036,854,775,808 到 9,223,372,036,854,775,807
- 适用于超大型表
```sql
CREATE TABLE large_data (
    id BIGSERIAL PRIMARY KEY,
    data JSONB
);
```

## 2. 通用唯一标识符 (UUID)

- 128位全局唯一标识符
- 不依赖序列生成器
- 分布式系统友好
```sql
CREATE TABLE distributed_table (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    content TEXT
);
```

## 3. 自然键 (Natural Keys)

使用业务中自然存在的唯一标识作为主键：
```sql
CREATE TABLE countries (
    country_code CHAR(2) PRIMARY KEY, -- ISO国家代码
    name VARCHAR(100)
);

CREATE TABLE products (
    product_sku VARCHAR(20) PRIMARY KEY, -- 库存单位代码
    description TEXT
);
```

## 4. 复合主键 (Composite Keys)

多列组合作为主键：
```sql
CREATE TABLE order_items (
    order_id INT,
    item_id INT,
    quantity INT,
    PRIMARY KEY (order_id, item_id)
);
```

## 5. 自定义序列 (Custom Sequences)

更灵活控制序列生成：
```sql
CREATE SEQUENCE custom_seq START WITH 100 INCREMENT BY 2;

CREATE TABLE custom_table (
    id INT PRIMARY KEY DEFAULT nextval('custom_seq'),
    details TEXT
);
```

## 6. 时间戳/日期主键

适用于时间序列数据：
```sql
CREATE TABLE time_series (
    event_time TIMESTAMPTZ PRIMARY KEY,
    value FLOAT
);
```

## 7. 哈希值主键

对内容生成哈希作为主键：
```sql
CREATE TABLE documents (
    content_hash BYTEA PRIMARY KEY, -- 存储SHA-256哈希
    content TEXT,
    created_at TIMESTAMPTZ
);
```

## 主键选择考虑因素
```
| 因素 | 序列整型 | UUID | 自然键 | 复合键 |
|------|---------|------|-------|-------|
| 大小 | 小 | 大 | 不定 | 不定 |
| 全局唯一性 | 否 | 是 | 可能 | 可能 |
| 分布式友好 | 否 | 是 | 可能 | 可能 |
| 可读性 | 低 | 低 | 高 | 中 |
| 插入性能 | 高 | 中 | 高 | 高 |
| 外键关系 | 简单 | 简单 | 复杂 | 复杂 |
| 索引效率 | 高 | 中 | 不定 | 不定 |
```
## 最佳实践
```
1. **默认选择**：大多数情况使用 `SERIAL` 或 `BIGSERIAL`
2. **分布式系统**：优先考虑 UUID
3. **历史数据**：时间序列数据可使用时间戳主键
4. **自然键**：确保真正唯一且不变时使用
5. **避免频繁更新**：主键值一旦建立不应修改
6. **外键考虑**：简单整型主键使外键关系更高效
```
## 修改主键示例

```sql
-- 删除现有主键
ALTER TABLE table_name DROP CONSTRAINT table_name_pkey;

-- 添加新主键
ALTER TABLE table_name ADD PRIMARY KEY (new_column);
```

选择合适的主键类型对数据库性能和数据完整性至关重要，应根据具体应用场景和数据特点决定。