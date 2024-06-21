```
cross_field会组合多个字段一起搜索 语句。
加上and后，表示每个词都必须出现。


```

```
跟most_field best_field对比 https://blog.csdn.net/weixin_40149557/article/details/122467573



best_fields则致力于返回尽可能精确的文档:   best_fields取子查询中算分最高的最为最终算分

most_field 让尽可能多的字段中有词汇的优先： most_fields则取所有子查询的算分的和做为最终算分
most_field加and后，要求所有的词都必须要出现在同一个字段。

cross_field会组合多个字段一起搜索
cross_field加and后，要求所有的词都必须在这个组合出来的集合字段中出现。



```