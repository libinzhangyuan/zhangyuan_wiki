[返回]()

### minimum_should_match

```
# 对bool中，可以加minimum_should_match
minimum_should_match: 4  表示bool下的should条件至少要匹配4个。

额外注意：
如果 bool 查询包含至少一个 should 子句，而没有 must 或 filter 子句，则minimum_should_match默认值为 1。
如果 bool 查询中同级子句中出现了 must 或者 filter 子句，则 minimum_should_match 的默认值将变为 0。
```


```
# query某个字段，可以加minimum_should_match
minimum_should_match: 75%  表示，此query中的多个关键词，至少要匹配75%

minimum_should_match: "3<90%"，表示至少需要匹配3个子句或总数的90%（以较大者为准）。 ??? 
https://blog.csdn.net/wlei0618/article/details/130577679

minimum_should_match: "-1" 最多不匹配的个数为一个

"-25%"  "3<90%" "2<-25% 9<-3" 
https://blog.csdn.net/weixin_38376791/article/details/132988721


并且，es背后其实会将query展开为 bool下单个should,或must(当query的条件为and时)， 即，展开为上面那个
见这个视频: https://www.bilibili.com/video/BV1jL411p78i?p=8&vd_source=12fa3a2f2f260d2e21c49b5cb6b91885




```