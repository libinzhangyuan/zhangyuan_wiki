[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的 hstore 类型

hstore 是 PostgreSQL 提供的一个扩展数据类型，用于存储键值对集合（类似于字典或哈希结构）。以下是关于 hstore 的详细介绍：

## 基本特性

- **键值存储**：存储形式为 `"键"=>"值"` 的集合
- **灵活性**：键和值都是文本字符串
- **非关系型**：适合存储半结构化数据
- **扩展需求**：需要先启用 hstore 扩展

## 启用 hstore 扩展

```sql
CREATE EXTENSION hstore;
```

## 基本语法

### 创建包含 hstore 列的表

```sql
CREATE TABLE products (
    id serial PRIMARY KEY,
    name text,
    attributes hstore
);
```

### 插入数据

```sql
-- 使用 hstore 字面量
INSERT INTO products (name, attributes) 
VALUES ('Laptop', '"color"=>"black", "cpu"=>"i7", "ram"=>"16GB"');

-- 使用 hstore 函数
INSERT INTO products (name, attributes) 
VALUES ('Phone', hstore(ARRAY['color', 'white', 'storage', '128GB']));
```

## 常用操作

### 查询操作

```sql
-- 获取所有键
SELECT akeys(attributes) FROM products;

-- 获取所有值
SELECT avals(attributes) FROM products;

-- 检查是否包含特定键
SELECT name FROM products WHERE attributes ? 'color';

-- 检查是否包含特定键值对
SELECT name FROM products WHERE attributes @> '"color"=>"black"'::hstore;
```

### 修改操作

```sql
-- 添加/更新键值对
UPDATE products 
SET attributes = attributes || '"weight"=>"1.5kg"'::hstore 
WHERE id = 1;

-- 删除键
UPDATE products 
SET attributes = delete(attributes, 'weight') 
WHERE id = 1;
```

## 索引支持

可以为 hstore 列创建索引以提高查询性能：

```sql
-- GIN 索引（适合各种查询）
CREATE INDEX idx_products_attributes ON products USING GIN(attributes);

-- GiST 索引（适合特定查询类型）
CREATE INDEX idx_products_attributes ON products USING GiST(attributes);
```

## 实际应用场景

- 存储产品可变属性
- 用户偏好设置
- 动态配置项
- 不需要固定列结构的元数据

## 注意事项

- 不适合需要复杂查询或高度结构化的数据
- 键和值都只能是字符串类型
- 对于更复杂的文档结构，可以考虑使用 JSON/JSONB 类型

hstore 提供了一种在关系型数据库中存储半结构化数据的轻量级解决方案，特别适合属性变化频繁的场景。