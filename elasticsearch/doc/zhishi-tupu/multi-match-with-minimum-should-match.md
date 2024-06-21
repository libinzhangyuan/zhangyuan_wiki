```

minimum_should_match，控制搜索结果的精准度，只有匹配一定数量的关键词的数据，才能返回

GET /forum/article/_search
{
  "query": {
    "multi_match": {
        "query":                "java solution",
        "type":                 "best_fields", 
        "fields":               [ "title^2", "content" ],
        "tie_breaker":          0.3,
        "minimum_should_match": "50%" 
    }
  } 
}


https://blog.csdn.net/qq_27384769/article/details/79645519
```