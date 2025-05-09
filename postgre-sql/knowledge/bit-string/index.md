[返回](/postgre-sql/knowledge/index)


# PostgreSQL 中的 Bit String 类型

PostgreSQL 提供了几种位串(Bit String)类型，用于存储位序列数据。这些类型在处理二进制标志、位掩码或需要位级操作的数据时非常有用。

## Bit String 类型

PostgreSQL 支持两种位串类型：

1. **bit(n)**: 固定长度的位串，n 是位数
2. **bit varying(n)** 或 **varbit(n)**: 可变长度的位串，最大长度为 n 位

如果不指定长度，`bit varying` 可以存储任意长度的位串。

## 位串字面量

位串常量可以用以下方式表示：

- `B'1010'`: 二进制表示法
- `X'1A'`: 十六进制表示法(每个十六进制数字代表4位)

## 基本操作

### 创建表

```sql
CREATE TABLE bit_example (
    id serial PRIMARY KEY,
    fixed_bits bit(8),
    variable_bits varbit(32)
);
```

### 插入数据

```sql
INSERT INTO bit_example (fixed_bits, variable_bits) 
VALUES (B'10101010', B'1100');
```

### 查询数据

```sql
SELECT * FROM bit_example;
```

## 位串函数和操作符

PostgreSQL 提供了丰富的位串操作：

### 连接操作

```sql
SELECT B'1001' || B'0110';  -- 结果为 B'10010110'
```

### 比较操作

```sql
SELECT B'1010' = B'1010';  -- 返回 true
SELECT B'1010' < B'1100';  -- 返回 true
```

### 位操作

```sql
SELECT B'1001' & B'0110';  -- 位与，结果为 B'0000'
SELECT B'1001' | B'0110';  -- 位或，结果为 B'1111'
SELECT B'1001' # B'0110';  -- 位异或，结果为 B'1111'
SELECT ~B'1001';           -- 位非，结果为 B'0110' (取决于位数)
```

### 移位操作

```sql
SELECT B'1001' << 2;  -- 左移2位，结果为 B'0100'
SELECT B'1001' >> 1;  -- 右移1位，结果为 B'0100'
```

### 其他函数

```sql
SELECT bit_length(B'1010');  -- 返回位数(4)
SELECT octet_length(B'10101010');  -- 返回字节数(1)
SELECT get_bit(B'1010', 2);  -- 获取第2位(从0开始)，返回1
SELECT set_bit(B'1010', 1, 0);  -- 将第1位设置为0，结果为 B'1000'
```

## 实际应用示例

### 存储权限位掩码

```sql
CREATE TABLE user_permissions (
    user_id int PRIMARY KEY,
    permissions bit(8)
);

-- 每位代表一种权限
INSERT INTO user_permissions VALUES 
(1, B'11110000'),  -- 有前4种权限
(2, B'00001111');  -- 有后4种权限

-- 检查是否有第3位权限
SELECT user_id FROM user_permissions 
WHERE get_bit(permissions, 2) = 1;
```

### 处理二进制数据

```sql
-- 将十六进制转换为位串
SELECT X'AB'::bit(8);  -- 结果为 B'10101011'

-- 将整数转换为位串
SELECT 42::bit(8);  -- 结果为 B'00101010'
```

## 注意事项

1. 位串索引从0开始(最左边是第0位)
2. 对于`bit(n)`类型，插入的值必须正好是n位
3. 位串操作通常比整数操作慢，只有在确实需要位级操作时才使用

位串类型在特定场景下非常有用，特别是需要精确位操作或存储紧凑二进制数据时。