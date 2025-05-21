### 命令
* [explain，及其所有选项](explain/index)
* [pg_repack, REINDEX 和 VACUUM](reindex-and-vacuum-pg-repack/index)

### 字段介绍
* [自增主键 ID (auto-incrementing primary key)](auto-incrementing-primary-key/index)
* [uuid](uuid/index)<br><br>
* [array](array/index) &nbsp;&nbsp; [cardinality 和 array_length](cardinality-and-array-length/index)
* [generated-column](generated-column/index)
* [TSVECTOR(Text Search Vector)](TSVECTOR/index)
* [bit string](bit-string/index)
* [复合类型(Composite Types)](composite-types/index)



### 索引
* [原理-堆和物理位置标识符(heaps and CTID)](heap-ctid/index)
* [原理-B-tree 索引](b-tree-index/index)
* [原理-主键(Primary Keys)和二级索引(Secondary Indexes)](primary-key-and-secondary-indexes/index)
* [原理-主键类型(Primary Keys types)](primary-keys-types/index)
<br><br>
* [排除约束(EXCLUSION 约束)](exclusion-constraints/index)
* [通用倒排索引 GIN(Generalized Inverted Index)](generalized-inverted-index/index)
* [GIST 索引(Generalized Search Tree)](gist/index)
* [外键约束 (Foreign Key Constraints)](forengn-key-constraints/index)
* [函数索引(Functional Index)](functional-index/index)
* [部分索引（Partial Index）](partial-index/index)
* [实现"允许NULL值但NULL值必须唯一"的约束](uniq-null-only-one/index)
* [NULL 值排序](null-order/index)
* [hash索引(hash index)](hash-index/index)


### 全文搜索
* [PostgreSQL 全文搜索类型详解](text-search/index)
* [全文搜索排名函数 ts_rank ts_rank_cd](ts-rank/index)
* [全文搜索向量更新SQL详解](/postgre-sql/knowledge/ts-rank/setweight/index)
* [全文搜索归一化选项详解](/postgre-sql/knowledge/ts-rank/guiyihua/index)
* [全文搜索排名值预计算方案](/postgre-sql/knowledge/ts-rank/recalc-rand-and-store/index)
* [全文搜索排名值预计算方案-通用查寻](/postgre-sql/knowledge/ts-rank/recalc-rand-and-store/general-search/index)
* [全文搜索排名值预计算方案-通用查寻 - 在SELECT查询中使用新闻网站通用排名](/postgre-sql/knowledge/ts-rank/recalc-rand-and-store/general-search/general-rank-by-news-website/index)
* [距离操作符，用于指定两个搜索词（或短语）之间的 相邻关系和距离 <-> or <3>](text-search/word-distance/index)
* [postgresql 的全文搜索 缺点是什么？](text-search/defect/index)

### 查寻
* [cross, inner, outer join](join/index)
* [子查寻 subquery](subquery/index)
* [lateral join](lateral-join/index)
* [rows from](rows-from/index)
* [Combining Query（组合查询） (union, union all, intersect, except)](combining-query/index)
* [子查询消除（Subquery Elimination）](subquery-elimination/index)
* [SEMI JOIN, anti join](semi-join-anti-join/index)
* [集合生成函数 (Set Generating Functions)](set-generating-function/index)
* [连接索引(Indexing Joins)](indexing-join/index)
* [GROUP BY 和 GROUPING SETS](grouping-set/index)

### 原理
* [WAL (Write-Ahead Logging，预写式日志)](wal/index)
* [原理-堆和物理位置标识符(heaps and CTID)](heap-ctid/index)
* [原理-B-tree 索引](b-tree-index/index)
* [Nested Loop Join, Hash Join, Merge Join](nested-loop-hash-merge-join/index)