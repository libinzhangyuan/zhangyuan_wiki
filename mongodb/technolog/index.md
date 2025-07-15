[返回](/mongodb/index)

### 设计 
* [MongoDB数据库设计经验法则](design-pricipal/index)
* [ 属性模式(Attribute Pattern)](attribute-pattern/index)
* [findAndModify 原子性操作](findAndModify-for-atom-modify/index)
* [双写应用层-比较偏](double-write/index)
<br><br><br>

### 分片相关
* [分片(Sharding)](sharding/index)
* [时间键范围分片：指定时间区间 vs 不指定时间区间](range-sharding-define-range-vs-no-define/index)
* [哈希分片](sharding-hash/index)
* [范围分片vs哈希分片](range-sharding-vs-hash-sharding/index)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
* [标签分片 - 不重要](tag-sharding/index)
* [复合分片](Compound-sharding/index)
* [分片与分桶详解](sharding-and-Bucketing/index)
* [跨分片查询](query-in-multi-sharding/index)
* [分片策略比较：按时间 vs 按用户ID](sharding-by-time-vs-by-user-id/index)
* [文章表， 要按照时间查寻最新的条目(次数多)，要根据栏目查寻最新的条目(次数最多)， 要根据文章id查寻条目(次数少)，栏目是数组[2, 4, 6]这种类型。 用什么分片](sharding-example-article/index)


### 变更流
* [变更流(Change Streams)](Change-Streams/index)
* [变更流应用场景](Change-Streams-usecase/index)
* [在Spring Boot中实现MongoDB变更流实时通知和提醒](Change-Streams-with-spring-boot/index)
* [Spring Boot中正确关闭MongoDB变更流监听](Change-Streams-with-spring-boot-how-to-close/index)
* [变更流(Change Stream)网络中断后的恢复机制](change-stream-disconnect-ctl-restore/index)


### 事务
* [mongodb事务-csdn](https://blog.csdn.net/u010003835/article/details/52912733)
<br>MongoDB 写关注(Write Concern)级别设置 https://chat.deepseek.com/a/chat/s/5de44bf4-84fd-4a75-84fc-baed7c6eb647
<br>MongoDB 读偏好(Read Preference)详解 https://chat.deepseek.com/a/chat/s/b2b71cc5-e328-4dda-8d38-6ef61b71071a
<br>两阶段提交



### 命令
* [Number类型](number/index)
* [Decimal 和 Double 数据类型介绍](decimal-double/index)
* [db.test.insertOne({ value: 5 }) 存储的是5 还是5.0](store-NumberInt-or-float/index)
* [显式类型指定函数](define-type-explicit/index)
<br><br>
* [查询 MongoDB 中 +8 时区某一天的数据](date/query-8-zoon)
* [Date Unix纪元(1970-01-01)以来的毫秒数](date/index)
* [new Date() 用法详解](new-date/index)
* [日期时间与时区解析](date-zone/index)
* [日期时区处理详解, $dateToString](date-ctl/index)
* [$date 操作符详解](dolar-date/index)
* [$dayOfMonth处理时区 - $dateToParts](date-to-parts/index)
* [setUTCHours 处理负数小时的情况](setUtcHours/index)
<br><br>
* [显式 $and 与隐式 AND 对比](explicit-and-vs-implicit-and/index)


### 查找
* [Cursor (游标)和Batch (批处理)](cursor-and-batch/index)
* [及时关闭游标](close-cursor/index) &nbsp; &nbsp;
* [Java try-with-resources](Java-try-with-resources/index)
* [MongoTemplate 在大集合中使用 skip 的性能问题](mongotemplate-skip-bad/index)
* [findAndModify 原子性操作](findAndModify-for-atom-modify/index)


### 优化
* [统计数量慢问题分析 count,estimatedDocumentCount,countDocuments](count/index)
* [count vs countDocuments 性能对比](count-vs-countDocuments/index)
* [索引](table-index/index)