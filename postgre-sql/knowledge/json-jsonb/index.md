[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的 JSON 和 JSONB 类型

PostgreSQL 提供了两种 JSON 数据类型：`json` 和 `jsonb`，它们都用于存储 JSON 格式的数据，但在存储方式和功能上有重要区别。

## 主要区别
```
| 特性        | JSON                          | JSONB (JSON Binary)             |
|------------|-------------------------------|---------------------------------|
| **存储格式**  | 文本格式（保留原始格式，包括空格） | 二进制分解格式（优化存储和查询） |
| **写入速度**  | 较快（不转换直接存储）          | 较慢（需要转换和优化）           |
| **查询速度**  | 较慢（每次需要解析）            | 较快（已优化存储结构）           |
| **索引支持**  | 有限                          | 完整支持                        |
| **体积**     | 通常较大                       | 通常较小                        |
```
## 基本用法

### 启用扩展（如果需要）

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
```

### 创建表

```sql
CREATE TABLE products (
    id serial PRIMARY KEY,
    name text,
    specs json,      -- 原始JSON格式
    specs_b jsonb    -- 二进制JSON格式
);
```

### 插入数据

```sql
INSERT INTO products (name, specs, specs_b)
VALUES (
    'Smartphone',
    '{"color": "black", "storage": "128GB", "features": ["GPS", "NFC"]}',
    '{"color": "black", "storage": "128GB", "features": ["GPS", "NFC"]}'
);
```

## 查询操作

### 基本查询

```sql
-- 获取整个JSON对象
SELECT specs FROM products WHERE id = 1;

-- 使用 -> 获取JSON对象的字段（返回JSON）
SELECT specs -> 'color' FROM products;

-- 使用 ->> 获取JSON对象的字段值（返回文本）
SELECT specs ->> 'color' FROM products;

-- 访问数组元素
SELECT specs -> 'features' -> 0 FROM products;  -- 返回第一个特性
```

### 条件查询

```sql
-- 查找颜色为黑色的产品
SELECT name FROM products WHERE specs ->> 'color' = 'black';

-- 检查是否包含特定键
SELECT name FROM products WHERE specs ? 'color';

-- 检查数组是否包含特定值 (jsonb专有)
SELECT name FROM products WHERE specs_b @> '{"features": ["NFC"]}';
```

## 修改操作

### JSONB 专有操作

```sql
-- 合并JSON对象
UPDATE products 
SET specs_b = specs_b || '{"weight": "150g"}'::jsonb 
WHERE id = 1;

-- 删除键
UPDATE products 
SET specs_b = specs_b - 'weight' 
WHERE id = 1;

-- 修改数组
UPDATE products 
SET specs_b = jsonb_set(specs_b, '{features,1}', '"Bluetooth"') 
WHERE id = 1;
```

## 索引支持

JSONB 支持多种索引类型：

```sql
-- 在特定路径创建GIN索引
CREATE INDEX idx_product_color ON products USING gin ((specs_b -> 'color'));

-- 对整个JSONB列创建GIN索引
CREATE INDEX idx_product_specs ON products USING gin (specs_b);

-- 创建B树索引用于特定值的快速查找
CREATE INDEX idx_product_storage ON products ((specs_b ->> 'storage'));
```

## 函数和操作符

PostgreSQL 提供了丰富的 JSON 函数：

```sql
-- 提取所有键
SELECT jsonb_object_keys(specs_b) FROM products;

-- 数组长度
SELECT jsonb_array_length(specs_b -> 'features') FROM products;

-- 类型转换
SELECT specs_b::text FROM products;
```

## 使用建议
```
1. **优先选择 jsonb** - 除非需要保留原始格式（如空格、键顺序等）
2. **大型文档** - 对于非常大的JSON文档，jsonb更节省空间
3. **频繁查询** - 需要高效查询时使用jsonb
4. **频繁更新** - 需要修改JSON内容时使用jsonb
```
JSONB 是 PostgreSQL 中处理 JSON 数据的推荐选择，它结合了 NoSQL 的灵活性和关系数据库的强大功能。