[返回](/postgre-sql/knowledge/index)

# PostgreSQL 浮点类型 (FLOAT) 详解

## FLOAT 类型概述

FLOAT 是 PostgreSQL 中的浮点数类型，用于存储近似数值数据，适合科学计算和不需要精确小数位的场景。

### 浮点类型家族

PostgreSQL 提供两种浮点类型：

```
- REAL (4字节): 单精度浮点数，约6位十进制数字精度
- DOUBLE PRECISION (8字节): 双精度浮点数，约15位十进制数字精度
- FLOAT(n): 可指定精度的浮点数，n为二进制精度位数
```

## FLOAT 基本用法

### 创建表使用 FLOAT 类型

```sql
CREATE TABLE scientific_data (
    id SERIAL PRIMARY KEY,
    measurement_name VARCHAR(100),
    single_precision REAL,
    double_precision DOUBLE PRECISION,
    custom_float FLOAT(24)
);

INSERT INTO scientific_data 
(measurement_name, single_precision, double_precision, custom_float)
VALUES
('重力加速度', 9.8, 9.80665, 9.80665),
('圆周率', 3.14159, 3.141592653589793, 3.141592653589793),
('光速(m/s)', 2.99792e8, 2.99792458e8, 2.99792458e8);
```

### 查询浮点数据

```sql
SELECT * FROM scientific_data;
```

可能的返回结果：

```
 id |  measurement_name  | single_precision | double_precision  |  custom_float  
----+--------------------+------------------+-------------------+----------------
  1 | 重力加速度        |              9.8 |         9.80665   |        9.80665
  2 | 圆周率            |         3.14159  | 3.141592653589793 | 3.141592653589793
  3 | 光速(m/s)         |        299792000 |    299792458      |   299792458
```

## FLOAT 与 NUMERIC 的比较

### 特性对比

```
特性                | FLOAT/DOUBLE PRECISION      | NUMERIC
--------------------+----------------------------+----------------------------
存储方式            | 二进制近似存储             | 精确十进制存储
精度                | 6-15位有效数字             | 用户定义(最高131072位)
计算速度            | 快                         | 慢
适用场景            | 科学计算、不需要精确的场景 | 财务、需要精确计算的场景
存储空间            | 固定(4或8字节)             | 可变(取决于精度)
```

### 精度差异示例

```sql
SELECT 
    0.1::REAL AS float_value,
    0.1::NUMERIC(10,2) AS numeric_value,
    0.1::REAL = 0.1::NUMERIC(10,2) AS equality_check;
```

可能的返回结果：

```
    float_value     | numeric_value | equality_check 
--------------------+---------------+----------------
 0.10000000149011612 |          0.10 | f
```

## 浮点运算示例

### 基本运算

```sql
SELECT 
    1.0::REAL / 3.0 AS real_division,
    1.0::DOUBLE PRECISION / 3.0 AS double_division,
    1.0::NUMERIC / 3.0 AS numeric_division;
```

可能的返回结果：

```
    real_division    |   double_division   |      numeric_division      
--------------------+---------------------+----------------------------
 0.3333333432674408 | 0.3333333333333333  | 0.3333333333333333333333333333
```

### 科学计数法

```sql
SELECT 
    6.02214076e23 AS avogadro_number,
    1.602176634e-19 AS elementary_charge;
```

可能的返回结果：

```
   avogadro_number    | elementary_charge  
----------------------+--------------------
 6.02214076e+23       | 1.602176634e-19
```

## 特殊浮点值

PostgreSQL 支持 IEEE 754 标准定义的特殊浮点值：

```sql
SELECT 
    'NaN'::REAL AS not_a_number,
    'Infinity'::REAL AS positive_infinity,
    '-Infinity'::REAL AS negative_infinity;
```

可能的返回结果：

```
 not_a_number | positive_infinity | negative_infinity 
--------------+-------------------+-------------------
          NaN |           Infinity |          -Infinity
```

## 浮点函数

PostgreSQL 提供了一系列用于浮点运算的函数：

### 常用数学函数

```sql
SELECT 
    SQRT(2.0) AS square_root,
    POWER(2.0, 10.0) AS power_result,
    LOG(100.0) AS logarithm,
    ROUND(3.14159265, 4) AS rounded;
```

可能的返回结果：

```
   square_root    | power_result |   logarithm    | rounded  
------------------+--------------+----------------+----------
 1.4142135623730951 |         1024 | 4.605170185988092 | 3.1416
```

### 随机数生成

```sql
SELECT RANDOM() AS random_float;
```

可能的返回结果：

```
    random_float     
---------------------
 0.7564354842316897
```

## 注意事项

1. **精度问题**：浮点数是近似存储，不适合需要精确计算的场景（如财务计算）
2. **比较操作**：浮点数比较应使用范围而非等号，例如 `ABS(a - b) < 0.00001`
3. **性能考虑**：FLOAT 运算通常比 NUMERIC 快，但精度较低
4. **特殊值处理**：NaN 和 Infinity 需要特殊处理
5. **类型转换**：隐式转换可能导致精度损失，应显式控制类型转换

## 最佳实践

1. 科学计算、工程应用使用 FLOAT/DOUBLE PRECISION
2. 财务、货币计算使用 NUMERIC
3. 避免在 WHERE 子句中直接比较浮点数
4. 明确指定需要的精度（FLOAT(n)）
5. 处理特殊值(NaN, Infinity)时添加条件检查