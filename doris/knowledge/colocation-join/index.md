```
https://doris.apache.org/zh-CN/docs/query/join-optimization/colocation-join/

建表
CREATE TABLE tbl (k1 int, v1 int sum)
DISTRIBUTED BY HASH(k1)
BUCKETS 8
PROPERTIES(
    "colocate_with" = "group1"
);

查看集群内已存在的 Group 信息
SHOW PROC '/colocation_group';

```