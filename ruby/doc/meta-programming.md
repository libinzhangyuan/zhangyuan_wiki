[原编程书籍介绍ruby-china](https://ruby-china.org/wiki/ruby-meta)<br><br>
[ [新手指南] Ruby 的 Block 以及 Proc.new 与 Lambda 的区别](https://ruby-china.org/topics/25197)<br>
[ruby ：为什么要搞出block，proc，lambda这三个玩意](https://zhuanlan.zhihu.com/p/89694746)<br><br>
[总结：Ruby中的@ % # $等各种千奇百怪的符号的含义等](https://blog.csdn.net/yy19890521/article/details/91376727)<br>
[Ruby instance_exec 和 instance_eval 有啥区别, class_exec 和 class_eval](https://ruby-china.org/topics/2286)

<br><br><br>

* 函数定义


[define_singleton_method在接收器中定义一个单例方法](https://vimsky.com/examples/usage/ruby-Object-method-i-define_singleton_method-rb.html)
```
# 下面这个定义的是类函数
class Example
  def initialize(test='hey')
    define_singleton_method :say_hello, lambda { test }
  end
end
Example.new.say_hello   -》 "hey"

# 但是下面这个又定义的是类的函数
Example.define_singleton_method(:who_am_i) do
  "I am"
end
Example.who_am_i   # ==> "I am: A"

# 第3种
guy = "Bob"
guy.define_singleton_method(:hello) { "#{self}: Hello there!" }
guy.hello    #=>  "Bob: Hello there!"
```

定义类函数
```
# 已验证
class Example
end
Example.send(:define_method, :say_hello) { "test" }
Example.new.say_hello
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