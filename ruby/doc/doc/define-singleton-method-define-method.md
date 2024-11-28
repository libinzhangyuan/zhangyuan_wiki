[返回](/ruby/doc/meta-programming)

在 Ruby 中，`define_method` 和 `define_singleton_method` 都是用于动态定义方法的方式，但它们的使用场景和目标对象有所不同。

### `define_method`
`define_method` 是 Ruby 中用于在类或模块中动态定义实例方法的方式。它允许你通过传入方法名（作为符号）和方法体（通常是一个块）来创建一个新的实例方法。

#### 语法：
```ruby
define_method(:method_name) do
  # method body
end
```

#### 示例：
```ruby
class MyClass
  define_method(:greet) do |name|
    "Hello, #{name}!"
  end
end

obj = MyClass.new
puts obj.greet("Alice")  # 输出: "Hello, Alice!"
```

在这个示例中，我们使用 `define_method` 动态地在 `MyClass` 中定义了一个实例方法 `greet`，该方法接受一个参数并返回一条问候信息。

### `define_singleton_method`
`define_singleton_method` 用于在单个对象的单例类（也称为 eigenclass）中动态定义一个方法。与 `define_method` 只能定义实例方法不同，`define_singleton_method` 是针对特定对象的方法定义，即它允许你给某个特定的对象添加方法，而不是给该类的所有实例添加方法。

#### 语法：
```ruby
object.define_singleton_method(:method_name) do
  # method body
end
```

#### 示例：
```ruby
obj = Object.new

obj.define_singleton_method(:greet) do |name|
  "Hello, #{name}!"
end

puts obj.greet("Alice")  # 输出: "Hello, Alice!"
```

在这个例子中，我们使用 `define_singleton_method` 给 `obj` 对象动态地定义了一个单例方法 `greet`，这个方法只能在 `obj` 上调用，而不会影响到其他 `Object` 类的实例。

### 总结对比：

| 特性                    | `define_method`                               | `define_singleton_method`                   |
|-------------------------|-----------------------------------------------|---------------------------------------------|
| 定义位置                | 定义实例方法（类的所有实例都可以调用）       | 定义单例方法（仅在特定对象上有效）         |
| 适用对象                | 类或模块                                     | 特定对象（例如：`obj`）                    |
| 使用场景                | 动态定义实例方法                             | 动态定义单例方法                           |

### 使用场景：
- **`define_method`** 适用于当你需要动态地为类的多个实例添加方法时。
- **`define_singleton_method`** 适用于当你只需要为特定对象（而不是类的实例）定义方法时。

这两种方法都非常强大，能够在运行时动态地修改类或对象的行为。