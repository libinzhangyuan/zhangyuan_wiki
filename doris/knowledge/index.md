[返回](/doris/index)

### 设计
* [FE BE broker](fe-be-broker/index)
* [宽表](wide-table/index)

* [Partition（分区）](partition/index) &nbsp;&nbsp;&nbsp; [Dynamic Partitioning(动态分区)](dynamic-partitioning/index)
* [Bucket(分桶)](bucket/index)
* [tablet(数据分片)](tablet/index)
* [Partition、Bucket 和 Tablet 详解](partition-bucket-tablet/index)
* [三种数据模型：Duplicate(明细模型)、Aggregate(聚合模型) 和 Unique(唯一性模型)](duplicate-aggregate-unique/index)

### sss
* [rollup和物化视图](rollup-and-materialized-view/index)


```
explain graph

rollup
查看某张表的rollup: SHOW ROLLUP FROM `your_database`.`your_table`;
查看所有的Rollup信息: SHOW ALTER TABLE ROLLUP;
创建： alter table user_costs add rollup rollup_cost_userid(user_id, cos);

物化视图 https://doris.apache.org/zh-CN/docs/query/view-materialized-view/materialized-view
查看当前表的物化视图及其结构： SHOW CREATE VIEW view_name; 
特定的物化视图，你可以使用 DESC view_name;   desc mv_test all;
查看所有:  SELECT * FROM information_schema.materialized_views; 
创建: CREATE MATERIALIZED VIEW my_materialized_view AS
       SELECT column1, column2 FROM my_table WHERE condition GROUP BY group_by_column ORDER BY order_column;
```

[各种索引](https://doris.apache.org/zh-CN/docs/table-design/index/index-overview)： &nbsp; &nbsp; &nbsp; [原则](index-yuanzhe)<br>
&nbsp; &nbsp;   点查索引:<br>
&nbsp; &nbsp; &nbsp; &nbsp;     前缀索引<br>
&nbsp; &nbsp;&nbsp; &nbsp;      倒排索引<br>
&nbsp; &nbsp;   跳数索引:<br>
&nbsp; &nbsp; &nbsp; &nbsp;     ZoneMap索引<br>
&nbsp; &nbsp; &nbsp; &nbsp;     BloomFilter索引<br>
&nbsp; &nbsp; &nbsp; &nbsp;     NGram BloomFilter索引<br>



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



bitmap_union(to_bitmap(user_id))




