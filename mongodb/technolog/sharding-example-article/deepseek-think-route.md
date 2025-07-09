[返回](/mongodb/technolog/sharding-example-article/index)

由于栏目字段是数组类型（即一篇文章可能属于多个栏目），这会对分片策略产生重大影响。在MongoDB中，数组字段作为分片键会导致以下问题：
1. 分片键不允许是数组类型（MongoDB分片键必须是单个字段或复合字段，且不能包含数组）。
2. 如果尝试使用数组字段作为分片键，MongoDB会报错。
因此，我们需要重新考虑分片策略。
### 问题分析
- **高频查询1**：按照时间查询最新条目（全局最新）
- **高频查询2**：根据栏目（数组）查询该栏目下的最新条目
- **低频查询**：根据文章ID查询
由于栏目是数组，无法直接作为分片键，我们需要寻找替代方案。
### 解决方案
#### 方案1：使用非数组字段作为分片键
如果文档中有其他非数组字段适合作为分片键，比如：
- `publishTime`（时间）
- `articleId`（文章ID）
- 或者添加一个新的字段如`primaryCategory`（主栏目）
但是根据查询模式，按栏目查询是最高频的，所以最好能利用分片键来优化按栏目的查询。
#### 方案2：使用多对多关联表（推荐）
由于一篇文章可能属于多个栏目，我们可以将文章和栏目的关系拆分成一个单独的集合（比如`article_category`），然后在这个关联集合上进行分片。
具体设计：
1. **文章集合（articles）**：存储文章基本信息，以`articleId`或`publishTime`作为分片键。
2. **文章-栏目关联集合（article_category）**：存储文章和栏目的对应关系，结构如下：
   ```json
   {
     "_id": ObjectId,
     "articleId": ObjectId, // 或者引用文章的ID
     "categoryId": Number, // 栏目ID
     "publishTime": Date   // 文章发布时间（冗余存储）
   }
   ```
   然后对`article_category`集合使用分片键`{categoryId: 1, publishTime: -1}`。
这样，按栏目查询最新文章就可以通过查询`article_category`集合实现：
```javascript
// 查询栏目6的最新10篇文章
db.article_category.find({categoryId: 6}).sort({publishTime: -1}).limit(10)
```
然后根据返回的`articleId`再去`articles`集合中获取文章详细信息。
#### 方案3：使用冗余的栏目条目
如果不想拆分集合，可以在文章文档中为每个栏目都复制一条记录（冗余存储），这样栏目字段就不再是数组，而是单个值。然后就可以使用`{category: 1, publishTime: -1}`作为分片键。
但是这样会导致数据量成倍增长（一篇文章属于N个栏目，就有N条记录），且更新文章时需要更新多条记录。
### 分片策略推荐
考虑到查询频率和性能要求，我推荐**方案2（关联表）**，因为它：
1. 解决了分片键不能是数组的问题。
2. 能够高效支持按栏目查询最新文章（高频操作）。
3. 数据扩展性好。
### 具体实现
#### 1. 文章集合（articles）分片策略
由于按文章ID查询是低频操作，而按时间查询最新条目是高频操作，我们可以选择：
- 分片键：`{publishTime: -1}` 或者 `{articleId: "hashed"}`
考虑到全局最新查询，建议使用：
```javascript
sh.shardCollection("blogDB.articles", {publishTime: -1})
```
#### 2. 文章-栏目关联集合（article_category）分片策略
```javascript
sh.shardCollection("blogDB.article_category", {categoryId: 1, publishTime: -1})
```
### 索引设计
#### 文章集合（articles）
```javascript
// 分片键自动索引：{publishTime: -1}
db.articles.createIndex({articleId: 1}, {unique: true}) // 按文章ID查询
```
#### 文章-栏目关联集合（article_category）
```javascript
// 分片键自动索引：{categoryId: 1, publishTime: -1}
// 另外，为了通过文章ID删除或更新关联关系（当文章删除时），可以创建：
db.article_category.createIndex({articleId: 1})
```
### 查询示例
#### 按栏目查询最新文章（高频）
```javascript
// 步骤1：从article_category中获取指定栏目的最新文章ID
const latestArticles = db.article_category.find({categoryId: 6})
                             .sort({publishTime: -1})
                             .limit(10)
                             .projection({articleId:1, _id:0})
                             .toArray();
// 步骤2：获取文章详细信息
const articleIds = latestArticles.map(article => article.articleId);
db.articles.find({_id: {$in: articleIds}}); // 注意：这里假设article_category中的articleId对应articles的_id
```
#### 按时间查询最新条目（全局）
```javascript
// 直接查询articles集合
db.articles.find().sort({publishTime: -1}).limit(10)
```
#### 按文章ID查询
```javascript
// 直接查询articles集合
db.articles.find({articleId: "ART-123"})
```
### 性能对比（方案2 vs 原始设计）
```
+-------------------------+---------------------+---------------------+
|        操作类型         |  原始设计（无分片键）|  关联表设计         |
+-------------------------+---------------------+---------------------+
| 按栏目查询最新文章      | 全分片扫描，慢      | 直接路由，非常快    |
| 按时间全局查询          | 较快（但可能有热点）| 快（分片键）        |
| 按文章ID查询            | 快（唯一索引）      | 快（唯一索引）      |
| 存储开销                | 低                  | 中（冗余存储关联关系）|
| 写入开销（新文章）      | 一次写入            | 1+N次写入（N=栏目数）|
+-------------------------+---------------------+---------------------+
```
### 结论
对于栏目是数组且需要高频按栏目查询最新文章的场景，推荐使用关联表（article_category）并采用`{categoryId: 1, publishTime: -1}`作为分片键。虽然增加了关联表，但能有效解决分片键限制，并优化了最频繁的查询操作。同时，文章集合采用`{publishTime: -1}`作为分片键，优化全局最新查询。