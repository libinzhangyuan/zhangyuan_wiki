[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的 bytea 类型

bytea 是 PostgreSQL 中的二进制数据类型，用于存储二进制字符串（即原始字节序列）。以下是关于 bytea 类型的关键信息：

## 基本特性

- **存储内容**：可以存储任意二进制数据，如图片、音频、PDF 文件等
- **大小限制**：理论上可存储最多 1GB 的数据（实际受 PostgreSQL 配置限制）
- **编码**：不同于文本类型，bytea 不关心字符编码问题

## 输入/输出格式

bytea 支持两种格式：

1. **十六进制格式**（PostgreSQL 9.0+ 默认）：
   ```sql
   E'\\xDEADBEEF'
   ```

2. **转义格式**（旧版默认）：
   ```sql
   E'\\000\\001\\002'
   ```

## 常用函数

- `encode(bytea, 'hex'/'base64'/'escape')` - 将 bytea 编码为文本格式
- `decode(text, 'hex'/'base64'/'escape')` - 将文本解码为 bytea
- `length(bytea)` - 获取二进制数据长度（字节数）
- `get_byte(bytea, offset)` - 获取指定位置的字节
- `set_byte(bytea, offset, newvalue)` - 设置指定位置的字节

## 示例用法

```sql
-- 插入二进制数据
INSERT INTO files (name, data) VALUES ('image.png', E'\\x89504E470D0A1A0A...');

-- 查询并编码为 base64
SELECT name, encode(data, 'base64') FROM files;

-- 转换文本为二进制
SELECT decode('54657374', 'hex');  -- 结果为 'Test' 的二进制表示
```

## 注意事项

- 大容量二进制数据可能影响数据库性能
- 对于非常大的文件，考虑使用 PostgreSQL 的大对象 (Large Object) 功能或外部存储
- 在应用程序中，通常需要将 bytea 与编程语言的数据类型进行转换

bytea 类型非常适合在数据库中存储小型到中型的二进制数据。