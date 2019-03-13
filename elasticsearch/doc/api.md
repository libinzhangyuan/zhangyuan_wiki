### 查询某个分析器的分析(分词)结果
```
GET /_analyze?analyzer=standard
Quick brown fox
```

* [match]() term查询


* [match_phrase](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/phrase-matching.html)短语匹配,短语查询<br>
  [slop](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/slop.html) in match_phrase 模糊化  <br>
  [position_increment_gap](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_multivalue_fields_2.html) = 100 多值字段的处理<br>
  [集合使用match和match_parse，使罗列尽量多，且让临近匹配的文档靠前](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/proximity-relevance.html)<br>
  [rescore  window_size  rescore_query](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_Improving_Performance.html) 取term查询的前面部分数据开展match_parse评分