[返回](/ruby/doc/meta-programming)

[下面的大部分来自本网页： Ruby 反射和内省](https://www.javascriptcn.com/ruby/67500d94a7f4f47d1aa33051.html)

```
# 获取类名
person = Person.new
puts person.class.name # 输出 "Person"

```

```
# 检查是否是某个类的实例
person = Person.new
puts person.is_a?(Person) # 输出 "true"
puts person.kind_of?(Person) # 输出 "true"
```


```
# 获取类的方法列表
// www.javascriptcn.com code example
class Person
  def initialize(name)
    @name = name
  end

  def say_hello
    puts "Hello, I'm #{@name}"
  end
end

# 实例方法
puts Person.instance_methods(false).inspect # 只输出本类的，不包含继承的 [:initialize]
puts Person.instance_methods(true).inspect # 包括继承的 [:say_hello, :initialize, ...]

# 类方法
Person.methods
Person.public_methods
```


```
# 获取方法的参数 parameters
class Person
  def initialize(name)
    @name = name
  end

  def say_hello(to)
    puts "Hello, #{to}, I'm #{@name}"
  end
end

puts Person.instance_method(:say_hello).parameters.inspect # 输出 "[[:req, :to]]"

```

```
# 获取对象的属性

class Person
  attr_accessor :name, :age

  def initialize(name, age)
    @name = name
    @age = age
  end
end

person = Person.new("Alice", 30)
puts person.instance_variables.inspect # 输出 [:@name, :@age]
```

```
# 获取属性值
person = Person.new("Alice", 30)
puts person.instance_variable_get(:@name) # 输出 "Alice"

设置属性值
person.instance_variable_set(:@name, "Bob")
```

```
# 获取类的常量 constants
class Person
  AGE_LIMIT = 18
end

puts Person.constants.inspect # 输出 [:AGE_LIMIT]
```

```
# 获取类的继承关系 和 祖先链
class Person
end

class Student < Person
end

puts Student.superclass.name # 输出 "Person"

puts Student.ancestors.map(&:name).inspect
```

