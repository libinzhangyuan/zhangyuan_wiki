[返回](/postgre-sql/knowledge/index)

# PostgreSQL 条件唯一索引详解

这个语句创建了一个特殊的条件唯一索引，用于实现"允许NULL值但NULL值必须唯一"的约束。让我们详细解析这个语句：

## 完整语句
```sql
CREATE UNIQUE INDEX idx_one_null_email 
ON users ((1)) 
WHERE email IS NULL;
```

## 各部分解析

### 1. `CREATE UNIQUE INDEX idx_one_null_email`
- 创建一个名为`idx_one_null_email`的唯一索引
- `UNIQUE`保证索引值唯一

### 2. `ON users ((1))`
- 在`users`表上创建索引
- `(1)`是一个常量表达式，所有行在这个索引中的值都是1
- 使用括号包裹是为了语法正确性

### 3. `WHERE email IS NULL`
- 这是一个部分索引(partial index)，只对满足条件的行创建索引
- 这里只索引`email`为NULL的行

## 工作原理

1. 当`email`为NULL时，索引会存储`(1)`这个值
2. 由于是UNIQUE索引，不允许有多个`(1)`值存在
3. 因此，这个索引确保表中最多只能有一行`email`为NULL的记录

## 实际效果

- 允许`email`列包含NULL值
- 但只允许存在一个NULL值（因为所有NULL值在索引中都映射为相同的`(1)`值）
- 非NULL的`email`值不受此索引影响

## 使用场景

这种索引特别适用于以下情况：
- 你想允许NULL值，但希望NULL值唯一（通常UNIQUE索引允许多个NULL值）
- 例如，用户表中email可选，但最多只能有一个用户可以不提供email

## 与传统UNIQUE索引的区别

普通UNIQUE索引：
```sql
CREATE UNIQUE INDEX idx_email ON users (email);
```
- 允许多个NULL值（SQL标准行为）
- 非NULL值必须唯一

这种特殊UNIQUE索引：
- 只允许一个NULL值
- 非NULL值不受影响（除非另有其他约束）

## 验证示例

```sql
-- 创建测试表
CREATE TABLE users (id serial, email text);

-- 创建特殊唯一索引
CREATE UNIQUE INDEX idx_one_null_email ON users ((1)) WHERE email IS NULL;

-- 可以插入多个非NULL email
INSERT INTO users (email) VALUES ('a@a.com'), ('b@b.com'); -- 成功

-- 可以插入一个NULL email
INSERT INTO users (email) VALUES (NULL); -- 成功

-- 尝试插入第二个NULL email会失败
INSERT INTO users (email) VALUES (NULL); -- 报错：重复键违反唯一约束
```

## 替代方案

另一种实现方式是使用条件约束（PostgreSQL 12+）：
```sql
CREATE TABLE users (
    id serial,
    email text,
    CONSTRAINT only_one_null_email CHECK (
        (SELECT COUNT(*) FROM users WHERE email IS NULL) <= 1
    )
);
```

但这种方法的性能不如部分索引好，因为每次插入都需要全表扫描检查。

## 性能考虑

- 这种索引非常轻量，只包含NULL值行
- 对非NULL值的插入/更新没有额外开销
- 比触发器或CHECK约束更高效

这个技巧展示了PostgreSQL索引系统的灵活性，能够通过创造性的方式实现特殊的业务约束。