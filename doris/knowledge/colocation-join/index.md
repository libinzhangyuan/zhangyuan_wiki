
```
https://doris.apache.org/zh-CN/docs/query/join-optimization/colocation-join/


CREATE TABLE tbl (k1 int, v1 int sum)
DISTRIBUTED BY HASH(k1)
BUCKETS 8
PROPERTIES(
    "colocate_with" = "group1"
);


```