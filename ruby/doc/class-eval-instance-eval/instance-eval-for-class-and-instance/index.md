[返回](/ruby/doc/class-eval-instance-eval/index)

# Ruby `instance_eval` 对类和实例的区别

`instance_eval` 是 Ruby 中一个强大的元编程方法，它的行为会根据调用对象是类还是实例而有所不同。

## 对实例使用 `instance_eval`

当对**实例对象**调用 `instance_eval` 时：

```ruby
class Person
  def initialize(name)
    @name = name
  end
end

p = Person.new("Alice")
p.instance_eval do
  puts @name          # 可以访问实例变量 => "Alice"
  def greet           # 定义的是单例方法（只对这个实例有效）
    "Hello, #{@name}!"
  end
end

puts p.greet         # => "Hello, Alice!"
p2 = Person.new("Bob")
# p2.greet           # 会报错，因为greet方法只定义在p上
```

特点：
1. 块中的 `self` 是接收者实例
2. 可以访问实例的实例变量
3. 定义的方法会成为该实例的单例方法

## 对类使用 `instance_eval`

当对**类对象**调用 `instance_eval` 时：

```ruby
class Person
end

Person.instance_eval do
  def species        # 定义的是类的单例方法（类方法）
    "Homo sapiens"
  end
end

puts Person.species  # => "Homo sapiens"
```

特点：
1. 块中的 `self` 是接收者类对象
2. 定义的方法会成为该类的单例方法（即类方法）
3. 不能访问实例变量（因为是在类级别）

## 关键区别总结

```

| 特性                | 对实例使用 instance_eval          | 对类使用 instance_eval            |
|---------------------|----------------------------------|-----------------------------------|
| `self` 指向          | 该实例对象                        | 该类对象                          |
| 定义的方法类型       | 实例的单例方法                    | 类的单例方法（类方法）            |
| 可访问的变量         | 实例变量                          | 类变量（@@）                      |
| 常用场景             | 为特定实例添加方法                | 定义类方法                        |

```

## 与 `class_eval` 的对比

对于类对象，还有一个类似的方法 `class_eval`（别名 `module_eval`）：

```ruby
Person.class_eval do
  def greet          # 定义的是实例方法
    "Hello!"
  end
end

p = Person.new
p.greet             # => "Hello!"
```

`class_eval` 会在类的上下文中执行，定义的是实例方法，而 `instance_eval` 定义的是类方法。

选择哪个方法取决于你想定义什么类型的方法以及你当前的操作对象是什么。