[返回]()

### minimum_should_match

```
# 对bool中，可以加minimum_should_match
minimum_should_match: 4  表示bool下的should条件至少要匹配几个。

```


```
# query某个字段，可以加minimum_should_match
minimum_should_match: 75%  表示，此query中的多个关键词，至少要匹配75%
"3<90%"，表示至少需要匹配3个子句或总数的90%（以较大者为准）。 ??? 

http://wiki.zy.cn/elasticsearch/doc/zhishi-tupu/minimum-should-match

并且，es背后其实会将query展开为 bool下单个should,或must(当query的条件为and时)， 即，展开为上面那个
见这个视频: https://www.bilibili.com/video/BV1jL411p78i?p=8&vd_source=12fa3a2f2f260d2e21c49b5cb6b91885




```