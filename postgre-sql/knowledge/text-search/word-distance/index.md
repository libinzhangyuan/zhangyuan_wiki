[返回](/postgre-sql/knowledge/index)

在 PostgreSQL 的全文搜索中，`<->` 是一个 **距离操作符**，用于指定两个搜索词（或短语）之间的 **相邻关系和距离**。具体含义如下：

---

### 1. **基本含义**
- `A <-> B` 表示：**词 A 和词 B 必须相邻出现，且顺序固定**（A 在前，B 在后）。
- 类似于短语搜索（英文中的 `"A B"`）。

#### 示例：
```sql
to_tsquery('hello <-> world')  -- 匹配 "hello world"，但不匹配 "world hello" 或 "hello the world"
```

---

### 2. **扩展用法：指定距离**
PostgreSQL 还支持更灵活的距离控制：
- `A <N> B` 表示：**A 和 B 之间最多允许间隔 (N-1) 个其他词**。
  - `N=1` 时等价于 `<->`（即必须相邻）。
  - `N>1` 时允许有间隔。

#### 示例：
```sql
to_tsquery('hello <1> world')   -- 等价于 hello <-> world（必须相邻）
to_tsquery('hello <2> world')   -- 匹配 "hello world" 或 "hello awesome world"
to_tsquery('hello <3> world')   -- 匹配最多间隔 2 个词（如 "hello the amazing world"）
```

---

### 3. **实际用例**
- **精确短语匹配**：
  ```sql
  -- 搜索包含 "machine learning" 的文档（严格相邻）
  SELECT title FROM documents 
  WHERE tsv @@ to_tsquery('machine <-> learning');
  ```

- **允许间隔的搜索**：
  ```sql
  -- 搜索 "quick" 和 "fox" 之间最多间隔 2 个词（如 "quick brown fox" 或 "quick lazy dog fox"）
  SELECT title FROM documents
  WHERE tsv @@ to_tsquery('quick <3> fox');
  ```

---

### 4. **注意事项**
1. **语言依赖**：分词效果受文本搜索配置（如 `english`、`simple`）影响。
2. **性能**：复杂的距离操作可能增加查询开销。
3. **与 `ts_rank_cd` 配合**：距离操作符会影响相关性评分（相邻的词通常得分更高）。

---

### 对比其他操作符

```
| 操作符     | 含义                  | 示例                          |
|------------|----------------------|------------------------------|
| `A & B`    | A 和 B 同时出现       | `hello & world`              |
| `A | B`    | A 或 B 出现           | `hello | world`              |
| `A <-> B`  | A 和 B 严格相邻       | `"hello world"` 短语         |
| `A <N> B`  | A 和 B 最多间隔 N-1 词 | `quick <3> fox`（间隔 2 词） |

通过灵活使用 `<->` 和 `<N>`，可以实现更精准的短语或近似短语搜索。

```