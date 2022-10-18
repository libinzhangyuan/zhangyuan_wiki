* 函数定义
定义类函数
```
# 待验证
class Example
  def initialize(test='hey')
    define_singleton_method :say_hello, lambda { test }
  end
end
```

定义函数
```
# 待验证
self.class.send(:define_method, :say_hello) { test }
```


定义instance函数   or
```
方案１：　已验证
c = C.new
def c.func1
  puts "instance method #{self}"
end
c.func1  # instance method #<C:0x0000000118ed08>

方案２： 已验证：
    string = "String"
    string.instance_eval do
      def new_method
        self.reverse
      end
    end

注意：方案１和２函数内部的作用域不一样．
```


```
判断类  https://www.jianshu.com/p/636fcacbff3f
instance_of? 方法用来判断对象是否是一个类的实例，会忽略继承。

判断类包括继承  https://www.jianshu.com/p/636fcacbff3f
is_a? kinde_of?


```