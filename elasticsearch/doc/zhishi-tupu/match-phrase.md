[返回](/elasticsearch/doc/zhishi-tupu/search)

```
和match查询类似，match_phrase查询首先解析查询字符串来产生一个词条列表。然后会搜索所有的词条，但只保留包含了所有搜索词条的文档，并且词条的位置要邻接。一个针对短语quick fox的查询不会匹配我们的任何文档，因为没有文档含有邻接在一起的quick和fox词条。
```


```
https://blog.csdn.net/sinat_29581293/article/details/81486761

```