[返回](/ruby/doc/class-eval-instance-eval/index)


# Ruby 中的 class_eval 和 instance_eval 全面介绍

在 Ruby 元编程中，`class_eval` 和 `instance_eval` 是两个核心方法，它们的行为会根据接收者的类型（普通对象或类/模块）而有所不同。

## 1. class_eval / module_eval

### 基本用法
`class_eval` 在类的上下文中执行代码块，主要用于定义实例方法。

```ruby
class MyClass; end

MyClass.class_eval do
  def instance_method
    puts "New instance method"
  end
end

MyClass.new.instance_method # => "New instance method"
```

### 特点
- 只能用于类或模块
- 在类的上下文中执行
- 可以定义实例方法
- 可以访问类的实例变量

## 2. instance_eval 的两种使用场景

`instance_eval` 的行为会根据接收者是普通对象还是类/模块而有所不同。

### 2.1 对普通对象使用 instance_eval

```ruby
obj = Object.new

obj.instance_eval do
  def singleton_method
    puts "Method only for this object"
  end
  
  @var = "instance variable"
end

obj.singleton_method # => "Method only for this object"
```

特点：
- 在对象的单例类中执行
- 可以定义单例方法（仅该对象可用）
- 可以访问对象的实例变量

### 2.2 对类/模块使用 instance_eval

```ruby
class MyClass; end

MyClass.instance_eval do
  def class_method
    puts "Class method added"
  end
end

MyClass.class_method # => "Class method added"
```

特点：
- 在类的单例类中执行
- 可以定义类方法（相当于 `def self.class_method`）
- 可以访问类的实例变量（注意：是类的实例变量，不是类的实例的实例变量）

## 3. 关键区别对比

```

| 场景                  | class_eval                      | instance_eval (对类)           | instance_eval (对对象)         |
|----------------------|--------------------------------|-------------------------------|-------------------------------|
| 接收者                | 类/模块                        | 类/模块                       | 任何对象                      |
| 执行上下文            | 类的内部                       | 类的单例类内部                 | 对象的单例类内部               |
| 定义的方法类型        | 实例方法                       | 类方法                        | 单例方法                      |
| 能否访问实例变量      | 能（类的实例变量）              | 能（类的实例变量）             | 能（对象的实例变量）           |
| 别名                 | module_eval                    | 无                            | 无                            |

```

## 4. 实际应用示例

### 动态添加类方法

```ruby
module MyModule
end

MyModule.instance_eval do
  def module_level_method
    "Available on MyModule itself"
  end
end

MyModule.module_level_method # => "Available on MyModule itself"
```

### DSL 实现示例

```ruby
class Configuration
  def self.setup(&block)
    new.instance_eval(&block)
  end
  
  def option(name, value)
    define_singleton_method(name) { value }
  end
end

config = Configuration.setup do
  option :timeout, 30
  option :verbose, true
end

config.timeout # => 30
config.verbose # => true
```

## 5. 注意事项

```
1. **作用域变化**：这些方法会改变当前 `self` 的指向
2. **性能考虑**：动态定义方法比静态定义有额外开销
3. **可读性**：过度使用会使代码难以理解
4. **类变量访问**：`class_eval` 可以访问类变量（`@@var`），而 `instance_eval` 不能
```

理解这些方法的细微差别对于 Ruby 元编程至关重要，特别是在框架开发和 DSL 设计中。