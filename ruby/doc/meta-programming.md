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



方案２： 待验证：
    string = "String"
    string.instance_eval do
      def new_method
        self.reverse
      end
    end
```