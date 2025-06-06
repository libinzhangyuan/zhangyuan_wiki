[返回](/doris/knowledge/index)

```
数据库表的索引设计和优化跟数据特点和查询很相关，需要根据实际场景测试和优化。虽然没有 "银弹"，Apache Doris 仍然不断努力降低用户使用索引的难度，用户可以根据下面的简单建议原则进行索引选择和测试。

    最频繁使用的过滤条件指定为 Key 自动建前缀索引，因为它的过滤效果最好，但是一个表只能有一个前缀索引，因此要用在最频繁的过滤条件上
    对非 Key 字段如有过滤加速需求，首选建倒排索引，因为它的适用面广，可以多条件组合，次选下面两种索引：

    有字符串 LIKE 匹配需求，再加一个 NGram BloomFilter 索引
    对索引存储空间很敏感，将倒排索引换成 BloomFilter 索引

    如果性能不及预期，通过 QueryProfile 分析索引过滤掉的数据量和消耗的时间，具体参考各个索引的详细文档

https://doris.apache.org/zh-CN/docs/table-design/index/index-overview 底部
```