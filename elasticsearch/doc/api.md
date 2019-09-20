### 分析用
```
查询某个分析器的分析(分词)结果
GET /_analyze?analyzer=standard
Quick brown fox

GET /my_index/_analyze?analyzer=autocomplete

查询详细细节，比如评分详细情况
GET /_search?explain 
{
   "query"   : { "match" : { "tweet" : "honeymoon" }}
}


当 explain 选项加到某一文档上时， explain api 会帮助你理解为何这个文档会被匹配，更重要的是，一个文档为何没有被匹配。
GET /us/tweet/12/_explain
{
   "query" : {
      "bool" : {
         "filter" : { "term" :  { "user_id" : 2           }},
         "must" :  { "match" : { "tweet" :   "honeymoon" }}
      }
   }
}

通过使用 analyze API 来分析单词 ，进而比较某字段的分析情况
GET /my_index/_analyze
{
  "field": "my_type.title",   
  "text": "Foxes"
}

[highlight高亮显示](https://www.elastic.co/guide/cn/elasticsearch/guide/current/highlighting-intro.html)

```

### 索引设置相关
* [index_analyzer,search_analyzer](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_index_time_search_as_you_type.html)对字段区分定义索引的分析器和搜索的分析器


* [自定义过滤器]()  [案例](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_index_time_search_as_you_type.html)

* [_alias](https://es.xiaoleilu.com/070_Index_Mgmt/55_Aliases.html)索引定义别名

* [_all](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/custom-all.html)自定义_all字段

* ["index_options": "docs"](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/scoring-theory.html)禁用词频统计

<br><br><br>


### 结构化搜索

* [term](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_finding_exact_values.html)精确值查找　constant_score
* [bool](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/combining-filters.html)bool过滤器，组合查询
* [查找多个精确值](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_finding_multiple_exact_values.html)  [多值精确相等](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_finding_multiple_exact_values.html)
* [范围过滤](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_ranges.html)
* [处理 Null 值](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_dealing_with_null_values.html)  exists  missing



### 查询相关

* [match term](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/term-vs-full-text.html)  词项精确匹配 (term)查询，即不会自动大小写转换等 [例子](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_how_match_uses_bool.html)


* [match_phrase](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/phrase-matching.html)短语匹配,短语查询<br> [operator and](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/match-multi-word.html) [minimum_should_match: 75%](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/match-multi-word.html)
  [boost](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_boosting_query_clauses.html)查询语句提升权重
  [slop](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/slop.html) in match_phrase 模糊化  <br>
  [position_increment_gap](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_multivalue_fields_2.html) = 100 多值字段的处理<br>
  [集合使用match和match_parse，使罗列尽量多，且让临近匹配的文档靠前](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/proximity-relevance.html)<br>
  [rescore  window_size  rescore_query](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_Improving_Performance.html) 取term查询的前面部分数据开展match_parse评分

* [shingles](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/shingles.html) bigrams字段 每个单词 以及它的邻近词 作为单个词项索引.搜索时chengdu university，可以让"chengdu university"得分比"chengdu first university"更高.

* [部分匹配](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/partial-matching.html)  部分匹配 WHERE text LIKE "%quick%" <br>
  [prefix](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/prefix-query.html) 前缀匹配 like "quick%" <br>
 [wildcard](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_wildcard_and_regexp_queries.html) 通配符 <br>
 [regexp](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_wildcard_and_regexp_queries.html) 正则表达式<br>
 [match_phrase_prefix](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_query_time_search_as_you_type.html) 查询时输入即搜索    "max_expansions": 50
 [edge_ngram](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_index_time_search_as_you_type.html) 利用autocomplete分词器，在索引时实现查询时输入即搜索功能

### 排序相关
[即考虑相关性又考虑时间先后]() [2](https://www.elastic.co/guide/cn/elasticsearch/guide/current/boosting-by-popularity.html) [3最详细的文章](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/query-dsl-function-score-query.html#function-decay)
[field_value_factor参数](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/query-dsl-function-score-query.html#function-field-value-factor)



### 算分相关
```
https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/scoring-theory.html
```