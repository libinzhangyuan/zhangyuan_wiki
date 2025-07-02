[返回](/mongodb/index)

### 设计 
* [MongoDB数据库设计经验法则](design-pricipal/index)
* [ 属性模式(Attribute Pattern)](attribute-pattern/index)
* [findAndModify 原子性操作](findAndModify-for-atom-modify/index)
<br>
* [分片(Sharding)](sharding/index)
* [哈希分片](sharding-hash/index)
<br>范围分片 标签分片 复合分片
<br>MongoDB 写关注(Write Concern)级别设置 https://chat.deepseek.com/a/chat/s/5de44bf4-84fd-4a75-84fc-baed7c6eb647
<br>MongoDB 读偏好(Read Preference)详解 https://chat.deepseek.com/a/chat/s/b2b71cc5-e328-4dda-8d38-6ef61b71071a


### 命令
* [Number类型](number/index)
* [Decimal 和 Double 数据类型介绍](decimal-double/index)
* [db.test.insertOne({ value: 5 }) 存储的是5 还是5.0](store-NumberInt-or-float/index)
<br><br>
* [Date Unix纪元(1970-01-01)以来的毫秒数](date/index)
* [new Date() 用法详解](new-date/index)
* [日期时间与时区解析](date-zone/index)
* [日期时区处理详解, $dateToString](date-ctl/index)
* [$date 操作符详解](dolar-date/index)
* [$dayOfMonth处理时区 - $dateToParts](date-to-parts/index)
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