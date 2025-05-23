[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的枚举类型 (ENUM)

PostgreSQL 提供了枚举类型 (ENUM) 来定义一组有限的静态值集合，非常适合存储固定选项的数据。

## 基本特性

- **预定义值集**：创建时必须指定所有可能的取值
- **类型安全**：确保列值只能是预定义的值之一
- **排序支持**：枚举值按照创建顺序有隐式排序
- **空间效率**：比文本类型更节省存储空间

## 创建枚举类型

```sql
-- 创建颜色枚举类型
CREATE TYPE color_enum AS ENUM (
    'red',
    'green',
    'blue',
    'yellow',
    'black',
    'white'
);

-- 创建尺寸枚举类型
CREATE TYPE size_enum AS ENUM (
    'XS',
    'S',
    'M',
    'L',
    'XL',
    'XXL'
);
```

## 使用枚举类型

### 创建表时使用枚举类型

```sql
CREATE TABLE products (
    id serial PRIMARY KEY,
    name text NOT NULL,
    color color_enum,
    size size_enum
);
```

### 插入数据

```sql
-- 有效插入
INSERT INTO products (name, color, size) 
VALUES ('T-Shirt', 'red', 'M');

-- 无效插入（会报错）
INSERT INTO products (name, color, size) 
VALUES ('Jacket', 'purple', 'S');  -- 'purple' 不在 color_enum 中
```

## 枚举操作

### 查询所有枚举值

```sql
SELECT enum_range(NULL::color_enum);
-- 结果: {red,green,blue,yellow,black,white}
```

### 排序行为

```sql
-- 枚举值按定义顺序排序
SELECT * FROM products ORDER BY size;
-- 顺序将是: XS, S, M, L, XL, XXL
```

### 类型转换

```sql
-- 文本转枚举
SELECT 'red'::color_enum;

-- 枚举转文本
SELECT color::text FROM products;
```

## 修改枚举类型

### 添加新值

```sql
ALTER TYPE color_enum ADD VALUE 'pink' AFTER 'red';
```

### 重命名类型

```sql
ALTER TYPE color_enum RENAME TO product_color_enum;
```

### 重命名值

```sql
-- PostgreSQL 12+ 支持
ALTER TYPE color_enum RENAME VALUE 'red' TO 'scarlet';
```

## 注意事项

1. **不可删除值**：PostgreSQL 不支持直接从枚举类型中删除值
2. **不可修改顺序**：枚举值的顺序在创建后不能更改
3. **性能考虑**：枚举类型比文本类型更高效，但灵活性较低
4. **迁移策略**：要删除或修改值，通常需要：
   - 创建新枚举类型
   - 修改表列使用新类型
   - 转换数据
   - 删除旧类型

## 使用场景

- 产品颜色、尺寸等固定属性
- 订单状态（如 'pending', 'shipped', 'delivered'）
- 用户角色（如 'admin', 'editor', 'viewer'）
- 任何有固定选项集的分类数据

## 与CHECK约束的对比

对于简单选项，也可以使用CHECK约束：

```sql
CREATE TABLE products (
    color text CHECK (color IN ('red', 'green', 'blue'))
);
```

**ENUM优势**：
- 类型安全（专门的类型）
- 更好的文档化
- 更清晰的数据库设计
- 更高效的存储和索引

枚举类型为固定值集合提供了类型安全、高效的存储解决方案，特别适合业务领域中的分类数据。