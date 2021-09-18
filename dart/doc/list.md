```
固定长度的list
list = List.filled(10, '');
向里面添加第11个元素时，会把报错
并且 list.length = 15 会报错

可变长度的list
list = [1, 2, 3]
list.length = 2 是可以调用的
print list; // [1, 2]

判断类型
if (list is String)
```