[返回](/elasticsearch/doc/zhishi-tupu/index)

搜索的类型： 

* match
* match_all
* match_phrase 
* [term](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/query-dsl-term-query.html)
* terms [minimum_should_match](minimum_should_match)
* [range](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/query-dsl-range-query.html)
* wildcard


bool

* [minimum_should_match](minimum_should_match)
* should (should - and)
* must
* must_not
* filter

constant_score [function_score](function-score)

content.keyword

调优
* explain
* [search_type=dfs_query_then_fetch](https://www.bilibili.com/video/BV1jL411p78i?p=10&vd_source=12fa3a2f2f260d2e21c49b5cb6b91885)

