# 容器


### 映射
```
HashTable HashMap ConcurrentHashMap 区别
http://blog.csdn.net/tgxblue/article/details/8479147
HashMap 非线程安全 新 继承于Map
  -> LinkedHashMap 可按照插入顺序或最后访问顺序遍历，支持删除最老元素,典型应用为实现FIFO和LRU(Least Recently Used)缓存
           https://www.cnblogs.com/lzrabbit/p/3734850.html
HashTable 线程安全 老 继承于Dictionary
ConcurrentHashMap 基于concurrentLevel划分出了多个Segment来对key-value进行存储，从而避免每次锁定整个数组，在默认的情况下，允许16个线程并发无阻塞的操作集合对象，尽可能地减少并发时的阻塞现象。

映射相关接口：
Map
  -> SortedMap 接口 保证排序
    -> NavigableMap 接口 支持最近匹配原则的条目检索
Dictionary

映射相关类：
Map
  -> AbstractMap abstract class
    -> EnumMap 使用数组来存放与枚举类型对应的值。这使得EnumMap的效率非常高 https://www.2cto.com/kf/201109/103574.html
    -> HashMap
      -> LinkedHashMap 支持按照插入顺序或最后访问顺序遍历
    -> WeakHashMap 所引用的对象可回收
    -> TreeMap  实现NavigableMap接口
    -> IdentityHashMap 按照引用相等性(==)来比较元素 https://www.cnblogs.com/zhxhdean/archive/2011/03/25/1995431.html
Dictionary
  -> HashTable 实现Map接口  方法是同步的


```



### List
```
接口：
Collection
- -> List
- -> Queue
-   -> Deque

类：
AbstractCollection implement Collection
- -> AbstractList  implement List
-   -> ArrayList     implement List
-   -> AbstractSequentialList
-     -> LinkedList implements List, Deque
-   -> Vector  implements List
-     -> Stack
BitSet


ArrayList Vector LinkedList 区别
http://www.aspku.com/kaifa/java/204937.html
ArrayList 内部是数组  支持高效的随机访问
Vector 老 内部是数组，并实现线程安全  继承于List  支持高效的随机访问
LinkedList 内部是链表

Stack 继承于Vector  老

Properties 存储String的键值对，用来存储配置文件

Arrays 直接对数组操作的一组函数
spliterator



```

Optional
```

```



### 容器的流操作  Streams
```
https://www.zybuluo.com/changedi/note/622375




```
