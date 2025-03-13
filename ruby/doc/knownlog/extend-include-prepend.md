[返回](/ruby/doc/knownlog/index)


[分享 Ruby 中 require,load,autoload,extend,include,prepend 的区别](https://ruby-china.org/topics/35350)



```

include
当一个类或者模块 include 了一个 module M 时，则该类或者模块就拥有了该 module M 的方法。


extend
当一个类使用 extend 时，相当于打开了该类的单件类，为其添加了单件方法。
当一个对象使用 extend时，相当于打开了该对象的单件类，为其添加了单件方法。


prepend 和 include 很像，当一个类 prepend 或 include 一个模块时，该模块中的方法会成为该类的实例方法。
二者的区别在于，模块在祖先链中的位置。使用 include 时，模块在包含它的类之上。如果是 prepend，则是在 prepend 它的类之下。而祖先链中位置的不同，决定了方法调用的顺序。



```


## extend 和singleton_class之间的关系

在 Ruby 中，`extend` 和 `singleton_class` 都与对象的单例方法（singleton methods）相关，但它们的作用和使用方式有所不同。以下是它们之间的关系和区别：

---

### 1. `extend`
- **作用**: 将模块中的方法作为单例方法（singleton methods）添加到对象或类中。
- **使用场景**: 当你希望将模块的方法添加到某个对象或类的单例类中时。
- **效果**: 
  - 如果对类使用 `extend`，模块的方法会成为类的类方法。
  - 如果对对象使用 `extend`，模块的方法会成为该对象的单例方法。

```ruby
module MyModule
  def my_method
    puts "This is a method from MyModule"
  end
end

# 对类使用 extend
class MyClass
  extend MyModule
end

MyClass.my_method  # 输出: This is a method from MyModule

# 对对象使用 extend
obj = Object.new
obj.extend(MyModule)
obj.my_method  # 输出: This is a method from MyModule
```

---

### 2. `singleton_class`
- **作用**: 每个对象都有一个单例类（singleton class），用于存储该对象特有的单例方法。
- **使用场景**: 当你需要直接操作对象的单例类时。
- **效果**: 可以通过 `singleton_class` 访问对象的单例类，并在其中定义方法或引入模块。

```ruby
obj = Object.new

# 在单例类中定义方法
obj.singleton_class.class_eval do
  def my_method
    puts "This is a singleton method"
  end
end

obj.my_method  # 输出: This is a singleton method
```

---

### `extend` 和 `singleton_class` 的关系
- **`extend` 的本质**: 当你对一个对象或类使用 `extend` 时，Ruby 实际上是将模块的方法添加到该对象或类的单例类中。
- **等价关系**: 
  - `obj.extend(MyModule)` 等价于 `obj.singleton_class.include(MyModule)`。
  - `MyClass.extend(MyModule)` 等价于 `MyClass.singleton_class.include(MyModule)`。

```ruby
module MyModule
  def my_method
    puts "This is a method from MyModule"
  end
end

# 使用 extend
obj = Object.new
obj.extend(MyModule)
obj.my_method  # 输出: This is a method from MyModule

# 使用 singleton_class.include
obj2 = Object.new
obj2.singleton_class.include(MyModule)
obj2.my_method  # 输出: This is a method from MyModule
```

---

### 总结
- `extend` 是一种便捷的方式，用于将模块的方法添加到对象或类的单例类中。
- `singleton_class` 是对象单例类的直接访问方式，可以用于更灵活的操作。
- `extend` 的底层实现依赖于 `singleton_class`，因此它们本质上是相关的。

如果你只是需要将模块的方法添加到对象或类中，使用 `extend` 更简洁；如果你需要直接操作单例类，可以使用 `singleton_class`。