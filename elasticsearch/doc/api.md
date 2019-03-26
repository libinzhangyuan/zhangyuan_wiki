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


```

### 索引设置相关
* [index_analyzer,search_analyzer](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_index_time_search_as_you_type.html)对字段区分定义索引的分析器和搜索的分析器


* [自定义过滤器]()  [案例](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_index_time_search_as_you_type.html)

* [_alias](https://es.xiaoleilu.com/070_Index_Mgmt/55_Aliases.html)索引定义别名

* ["index_options": "docs"](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/scoring-theory.html)禁用词频统计

<br><br><br>


### 查询相关

* [match]() term查询


* [match_phrase](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/phrase-matching.html)短语匹配,短语查询<br>
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

### 算分相关
```
https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/scoring-theory.html
```