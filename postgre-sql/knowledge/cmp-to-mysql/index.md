[返回](/postgre-sql/knowledge/index)

# MySQL 中的 TEXT 类型使用策略

与 PostgreSQL 不同，MySQL 在处理文本类型时有自己的一些最佳实践。以下是 MySQL 中文本类型的使用建议：

## MySQL 文本类型选择策略

1. **优先考虑 VARCHAR 而不是 TEXT**（与 PostgreSQL 不同）
   - MySQL 的 VARCHAR 性能通常优于 TEXT
   - VARCHAR 支持完整的索引（TEXT 需要前缀索引）
   - 对于 ≤64KB 的文本，VARCHAR 是更好的选择

2. **TEXT 类型的使用场景**：
   - 存储超过 65,535 字符的内容（需要 MEDIUMTEXT 或 LONGTEXT）
   - 存储非结构化大文本（如文章内容、日志等）
   - 不确定长度的长文本

## MySQL 各文本类型比较
```
| 类型 | 最大长度 | 存储特性 | 适用场景 |
|------|---------|----------|----------|
| CHAR | 255字符 | 固定长度 | 短且长度固定的字符串（如国家代码） |
| VARCHAR | 65,535字节（实际约16,383字符，utf8mb4） | 可变长度 | 大多数常规字符串 |
| TINYTEXT | 255字符 | 可变长度 | 小段文本 |
| TEXT | 65,535字符 | 可变长度 | 文章、描述等中等长度文本 |
| MEDIUMTEXT | 16,777,215字符 | 可变长度 | 长篇内容、文档 |
| LONGTEXT | 4,294,967,295字符 | 可变长度 | 极大文本内容 |
```
## 实际开发建议

1. **常规字符串**：优先使用 VARCHAR(255) 或适当长度
   - `VARCHAR(255)` 是 MySQL 的"甜点"长度
   - 对于明确知道最大长度的字段，使用具体长度（如手机号 VARCHAR(20)）

2. **大文本内容**：
   - 预计会超过 16KB：使用 TEXT
   - 超过 16MB：使用 MEDIUMTEXT
   - 极大文本（如整本书）：LONGTEXT

3. **性能考虑**：
   - VARCHAR 支持完整索引，TEXT 只能前缀索引（前768字节）
   - TEXT/BLOB 列可能导致临时表使用磁盘而非内存

4. **编码影响**：
   - 使用 utf8mb4 时，每个字符占4字节
   - VARCHAR(255) 实际占用 ≤1020字节

## 与 PostgreSQL 的区别

PostgreSQL 推荐优先使用 TEXT 的原因：
- TEXT 在 PostgreSQL 中性能与 VARCHAR 几乎无差别
- 不需要预先指定长度
- 更简单的API

而 MySQL 由于历史架构原因，VARCHAR 和 TEXT 有更明显的性能差异，因此推荐策略不同。

**结论**：在 MySQL 中，常规字符串优先使用 VARCHAR，真正的大文本内容才使用 TEXT 类型。