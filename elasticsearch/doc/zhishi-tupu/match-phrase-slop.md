```
近似匹配, 邻接匹配
比如slop 15 这种大的数字，能够匹配离得近的.

```


```
官方，查看参数方便
https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query-phrase.html

```

### 优化
```
https://blog.csdn.net/qq_42383787/article/details/99967622

性能代价较高：
短语查询和邻近查询都比简单的 query 查询代价更高 。 一个 match 查询仅仅是看词条是否存在于倒排索引中，而一个 match_phrase 查询是必须计算并比较多个可能重复词项的位置。

match query的性能比match_phrase和proximity match（有slop）要高很多，因为后两者都要计算position的距离。

match query比match_phrase的性能要高10倍，比proximity match（带slop的）性能要高20倍。

优化方案rescore: https://blog.csdn.net/qq_42383787/article/details/99967622

```