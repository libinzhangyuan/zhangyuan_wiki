```


1、介绍

match_bool_prefix 查询内部将输入文本通过指定analyzer分词器处理为多个term，然后基于这些个term进行bool query，
除了最后一个term使用前缀查询 其它都是term query。
查询语句：



GET /_search
{
    "query": {
        "match_bool_prefix" : {
            "message" : "quick brown f"
        }
    }
}
类似于：



GET /_search
{
    "query": {
        "bool" : {
            "should": [
                { "term": { "message": "quick" }},
                { "term": { "message": "brown" }},
                { "prefix": { "message": "f"}}
            ]
        }
    }
}
和 match_phrase_prefix query重要的不同点是，match_phrase_prefix query前缀匹配是以短语为最小粒度进行的，而 match_bool_prefix 如果不对相关度进行限制的话，它会匹配更多的内容。

2、操作

参数
参数	说明
analyzer	指定terms文本分词器，默认是用mapping阶段指定的分词器
minimum_should_match	指定匹配度，可以是[0,1]的小数，也可以是百分比
operator	指定多个term之间的匹配方式，and或者or



```