[back](../Home)

http://train.usaco.org/usacogate  libinzh1 th55bcz


* [排序算法稳定性](https://baike.baidu.com/item/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95%E7%A8%B3%E5%AE%9A%E6%80%A7)
* [算法的复杂度的稳定性](https://zhidao.baidu.com/question/29388495.html)


### 可视化
* [可视化](http://blog.jobbole.com/72850/)


* [平衡二叉树、B树、B+树、B*树的差别](https://zhuanlan.zhihu.com/p/27700617)

* [二叉堆（最大堆/最小堆）的详解](https://www.cnblogs.com/skywang12345/p/3610187.html)
* [二叉堆的作用](https://zhidao.baidu.com/question/646662298629003565.html)


## 非比较排序

* [计数排序(鸽巢排序)](https://segmentfault.com/a/1190000003054515)

* [基数排序](https://segmentfault.com/a/1190000003054515) [2](https://www.cnblogs.com/Braveliu/archive/2013/01/21/2870201.html) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   [为什么低位先排](https://www.zhihu.com/question/27021728)

* [桶排序](https://baike.baidu.com/item/%E6%A1%B6%E6%8E%92%E5%BA%8F/4973777) [2](https://segmentfault.com/a/1190000003054515) [3](https://blog.csdn.net/asce1885/article/details/5620410) Bucket Sort 有时也称为盒子排序Bin Sort

## 比较排序

### 交换排序
冒泡排序<br>
[梳排序](https://blog.csdn.net/u010647471/article/details/50170825) <br>
梳排序和希尔排序很类似。希尔排序是在直接插入排序的基础上做的优化，而梳排序是在冒泡排序的基础上做的优化。也是想希尔排序一样，将待排序序列通过增量分为若干个子序列，然后对子序列进行一趟冒泡排序，一步步减小增量，直至增量为1。所以梳排序的最后一次排序是冒泡排序。 

双向冒泡排序（鸡尾酒排序）<br/>

快速排序<br/>
[快速排序和变种快排](https://blog.csdn.net/LYhani82475/article/details/79702914)
随机化快排 平衡快排 外部快排 三路基数快排


### 选择排序
* [选择排序](https://baike.baidu.com/item/%E9%80%89%E6%8B%A9%E6%8E%92%E5%BA%8F)
* [树形选择排序](https://www.cnblogs.com/mengdd/archive/2012/11/27/2791412.html)
* [堆排序](http://bubkoo.com/2014/01/14/sort-algorithm/heap-sort/)  [2](https://www.cnblogs.com/skywang12345/p/3610187.html) [3](https://zhidao.baidu.com/question/646662298629003565.html) Heap sort


### 插入排序
* [直接插入排序](https://baike.baidu.com/item/%E6%8F%92%E5%85%A5%E6%8E%92%E5%BA%8F)
* [希尔排序](https://www.cnblogs.com/jingmoxukong/p/4303279.html) &nbsp;&nbsp;&nbsp; [2](https://www.zhihu.com/question/24637339)
希尔排序的实质就是分组插入排序，该方法又称缩小增量排序
[专家们提倡，几乎任何排序工作在开始时都可以用希尔排序，若在实际使用中证明它不够快，再改成快速排序这样更高级的排序算法.](https://baike.baidu.com/item/%E5%B8%8C%E5%B0%94%E6%8E%92%E5%BA%8F)

### 合并排序(归并排序)
[归并排序](https://www.cnblogs.com/chengxiao/p/6194356.html)


### 其他排序
* [圈排序](https://en.wikipedia.org/wiki/Cycle_sort) Cycle sort  没有什么实用价值

* [地精排序](http://www.voidcn.com/article/p-dllqolqe-ph.html) gnome sorting 没有什么实用价值



# 字符串排序

* [低位优先的字符串排序](https://www.cnblogs.com/sun-haiyu/p/7877651.html)
首先待排序的字符串长度均相同，设为W，从右向左以每个字符作为关键字，用计数排序法将字符串排序W次。

* [高位优先的字符串排序MSD](https://www.cnblogs.com/sun-haiyu/p/7877651.html)
