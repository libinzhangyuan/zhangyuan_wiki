[返回](/elasticsearch/doc/zhishi-tupu/index)

搜索的类型： 

* match
* match_all
* [match_phrase](match-phrase) &nbsp;&nbsp;&nbsp; [match_phrase with slop](match-phrase-slop)
* [term](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/query-dsl-term-query.html)
* terms
* [range](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/query-dsl-range-query.html)
* [minimum_should_match](minimum-should-match)
* [multi_match](multi-match) best_fields、most_fields、[cross_fields](cross-fields) [(cross_fields with and)](cross-fields-with-and)  [tie_breaker](multi-match-with-tie-breaker) [minimum_should_match](multi-match-with-minimum-should-match)
* [bool_prefix](bool-prefix) &nbsp;&nbsp;&nbsp; tiebreaker copy_to

* [prefix](prefix)  [wildcard](wildcard)  [regexp](regexp)
* [match_phrase_prefix](match-phrase-prefix)
* [match_bool_prefix](match-bool-prefix)  [search_as_you_type](search-as-you-type)
* [Joining queries (nested query, has_child and has_parent queries)](https://www.elastic.co/guide/en/elasticsearch/reference/current/joining-queries.html)

* [dis_max](dis-max)  [tie_breaker](dis-max-with-tie-breaker)
* [geo query](geo-query)
* [sharp query](sharp-query)

bool

* should (should - and)
* must
* must_not
* filter
* [minimum_should_match](minimum_should_match)

constant_score [function_score](function-score)

[Disjunction max query](disjunction-max-queryedit)

[Boosting query](boosting-query)

content.keyword


调优

* explain
* [search_type=dfs_query_then_fetch](https://www.bilibili.com/video/BV1jL411p78i?p=10&vd_source=12fa3a2f2f260d2e21c49b5cb6b91885)
* [rescore](rescore)
