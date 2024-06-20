```
GET zfc-doc-000006/_search
{
  "query": {
    "match": {
      "content": "实战"
    }
  },
  "rescore": {
    "query": {
      "rescore_query":{
        "match":{
          "title":"MySQL"
        }
      },
      "query_weight" : 0.7,
      "rescore_query_weight" : 1.2
    },
    "window_size": 50
  }
}


理解 query_weight rescore_query_weight
上述查询语句的意思就是查询content字段中包含”实战“的文档，权重为0.7。并对文档中title为MySQL的文档增加评分，权重为1.2，window_size为50，表示取分片结果的前50进行重新算分

https://segmentfault.com/a/1190000044140174
```



```

利用rescore优化match_phrase
https://blog.csdn.net/qq_42383787/article/details/99967622

利用resore引入点赞数的影响
https://blog.csdn.net/w1014074794/article/details/120496215
```