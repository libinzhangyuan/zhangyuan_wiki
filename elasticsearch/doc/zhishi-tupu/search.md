[返回](/elasticsearch/doc/zhishi-tupu/index)

搜索的类型： 
match  match_all match_phrase 
[term](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/query-dsl-term-query.html)
terms [minimum_should_match](minimum_should_match)
[match]()
[range](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/query-dsl-range-query.html)
wildcard

bool [minimum_should_match](minimum_should_match)
should must filter must_not

constant_score function_score

content.keyword


explain
```




    "function_score": {
      "query": {
        "bool": {
          "should": [#{shouldStr}],
          "filter": [#{filterStr}]
        }
      },
      "functions": [
        {
          "gauss": {
            "publishTime": {
              "scale": "300d",  // 衰减的时间尺度，例如30天
              "offset": "0d",   // 衰减的偏移量，可以为负值
              "decay": 0.8      // 衰减因子，0到1之间，值越大衰减越缓慢
            }
          },
          "weight": 5           // 权重值，可以调整该函数对最终评分的影响
        }
        ],
        "score_mode": "multiply", // 评分模式，可以是multiply, sum, avg, first或min
        "boost_mode": "multiply"  // 增强模式，可以是multiply或replace
    }

```