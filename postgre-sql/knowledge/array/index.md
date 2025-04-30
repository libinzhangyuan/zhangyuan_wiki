[返回](/postgre-sql/knowledge/index)

在 PostgreSQL 中，**数组（Array）** 是一种内置的数据类型，允许在单个字段中存储多个相同类型的值。数组可以是一维或多维的，支持各种基础类型（如整数、文本等）和自定义类型。以下是关于 PostgreSQL 数组的详细介绍和用法示例：

---

### **1. 数组类型的基本操作**
#### **创建包含数组列的表**
```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    tags TEXT[],  -- 文本数组
    prices NUMERIC(10,2)[]  -- 数值数组
);
```

#### **插入数组数据**
```sql
-- 使用花括号 {} 定义数组
INSERT INTO products (name, tags, prices)
VALUES (
    '笔记本电脑',
    '{"电子产品", "便携", "高性能"}',
    '{5999.99, 5499.99}'  -- 不同价格版本
);
```

#### **查询数组**
```sql
-- 查询所有标签包含"电子产品"的商品
SELECT name FROM products WHERE '电子产品' = ANY(tags);

-- 使用数组下标（从1开始）
SELECT name, tags[1] AS primary_tag FROM products;
```

---

### **2. 数组常用函数和操作符**

```
#### **操作符**
| 操作符       | 描述                  | 示例                          |
|--------------|-----------------------|-------------------------------|
| `=`          | 数组是否相等          | `SELECT ARRAY[1,2] = ARRAY[1,2];` |
| `@>`         | 是否包含              | `SELECT ARRAY[1,2] @> ARRAY[1];`  |
| `<@`         | 是否被包含            | `SELECT ARRAY[1] <@ ARRAY[1,2];`  |
| `&&`         | 是否有重叠            | `SELECT ARRAY[1,2] && ARRAY[2,3];`|

#### **常用函数**
| 函数                          | 描述                     |
|-------------------------------|--------------------------|
| `array_append(arr, elem)`     | 向数组末尾添加元素       |
| `array_length(arr, dim)`      | 获取数组长度             |
| `array_position(arr, elem)`   | 返回元素首次出现的位置   |
| `unnest(arr)`                 | 将数组展开为行           |

```
**示例：**
```sql
-- 展开数组为多行
SELECT name, unnest(tags) AS tag FROM products;

-- 修改数组
UPDATE products 
SET tags = array_append(tags, '新品') 
WHERE id = 1;
```

---

### **3. 多维数组**
PostgreSQL 支持多维数组（如矩阵），但每维长度必须一致：
```sql
-- 创建二维数组（3x2矩阵）
CREATE TABLE matrices (
    id SERIAL PRIMARY KEY,
    matrix INTEGER[][]
);

-- 插入数据
INSERT INTO matrices (matrix) 
VALUES ('{{1,2}, {3,4}, {5,6}}');

-- 查询第二行第一列的值
SELECT matrix[2][1] FROM matrices;  -- 返回 3
```

---

### **4. 注意事项**
1. **索引优化**：  
   对数组列使用 `GIN` 索引可加速 `ANY`、`@>` 等操作：
   ```sql
   CREATE INDEX idx_tags ON products USING GIN(tags);
   ```
2. **NULL 处理**：  
   数组中的 `NULL` 与其他元素的 NULL 语义不同，需用 `IS NULL` 判断。
3. **性能**：  
   频繁修改大型数组可能影响性能，考虑规范化设计（如关联表）。

---

### **5. 实际应用场景**
- **标签系统**：存储文章的多个标签。
- **多值属性**：如商品的多个颜色、尺寸。
- **时序数据**：存储传感器的一组连续读数。

---

如果需要更具体的数组操作（如数组聚合、JSON 与数组转换等），可以进一步提问！