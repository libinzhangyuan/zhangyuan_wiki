```
multi_match 使用best_fields  致力于返回尽可能精确的文档:   best_fields取子查询中算分最高的最为最终算分
multi_match 用best_fields时，附带上tie_breaker时，会加权其他field的得分, 增加一点合理性。
通过tie_breaker来加权其他field的得分



https://blog.csdn.net/qq_27384769/article/details/79645519


```