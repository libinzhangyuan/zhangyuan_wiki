[返回](/doris/index)

```
explain graph

rollup
查看某张表的rollup: SHOW ROLLUP FROM `your_database`.`your_table`;
查看所有的Rollup信息: SHOW ALTER TABLE ROLLUP;
创建： alter table user_costs add rollup rollup_cost_userid(user_id, cos);

物化视图
查看单个： SHOW CREATE VIEW view_name; 
查看所有:  SELECT * FROM information_schema.materialized_views; 
```

各种join:<br>
 &nbsp;&nbsp;&nbsp;&nbsp; 
[join分析：shuffle hash join、broadcast hash join](https://www.cnblogs.com/tgzhu/p/15211820.html) <br>
 &nbsp;&nbsp;&nbsp;&nbsp; 
[优化原理](https://doris.apache.org/zh-CN/docs/query/join-optimization/doris-join-optimization/)<br>
 &nbsp;&nbsp;&nbsp;&nbsp; 
broadcast join &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
shuffle join &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
[Bucket Shuffle](bucket-shuffle) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
[colocation join](colocation-join/index) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
[sort merge join](sort-merge-join)
