[返回](/ruby/doc/meta-programming)


在 Ruby 中，**类变量**和**实例变量**是两种常见的变量类型，用于管理不同范围的数据。以下是它们的详细介绍：  

---

## 1. **实例变量 (`@variable`)**
### 定义与特点：
- 以 **`@`** 开头，例如 `@name`。
- 属于 **特定对象实例**，每个对象的实例变量是独立的。
- 默认值为 `nil`，如果未初始化直接访问，不会报错，而是返回 `nil`。
- 通常用来存储对象的状态或属性。

### 使用场景：
- 用于封装每个对象独立的数据，常用于类的初始化方法中。
- 配合 getter 和 setter 方法（或 `attr_accessor` 等）操作对象属性。

### 示例代码：
```ruby
class Person
  attr_accessor :name, :age

  def initialize(name, age)
    @name = name
    @age = age
  end

  def greet
    "Hi, I'm #{@name} and I'm #{@age} years old."
  end
end

person1 = Person.new("Alice", 30)
person2 = Person.new("Bob", 25)

puts person1.greet # Hi, I'm Alice and I'm 30 years old.
puts person2.greet # Hi, I'm Bob and I'm 25 years old.
```

---

## 2. **类变量 (`@@variable`)**
### 定义与特点：
- 以 **`@@`** 开头，例如 `@@counter`。
- 属于 **整个类**，以及该类的所有实例共享。
- 可用于存储与类相关的全局状态。
- 初次访问未初始化的类变量时，会抛出 `NameError`。

### 使用场景：
- 用于存储所有对象的共享数据，如计数器、配置信息等。

### 示例代码：
```ruby
class Person
  @@population = 0

  def initialize(name, age)
    @name = name
    @age = age
    @@population += 1
  end

  def self.population
    @@population
  end
end

person1 = Person.new("Alice", 30)
person2 = Person.new("Bob", 25)

puts Person.population # 2
```

---

## 主要区别：
| 特性                 | **实例变量 (`@`)**                | **类变量 (`@@`)**                         |
|----------------------|----------------------------------|-------------------------------------------|
| 所属范围             | 单个对象实例                     | 整个类及其所有实例                       |
| 数据是否共享         | 不共享，独立于其他对象            | 共享，所有实例都可以访问和修改            |
| 使用场景             | 对象的状态或属性                 | 类的全局状态或共享信息                   |
| 定义位置             | 在实例方法或类中初始化            | 通常在类中定义，适合存储类级别的数据      |

---

## 注意事项：
- **类变量容易引发问题**：
  - 因为所有实例共享类变量，误修改可能会影响其他实例的行为。
  - Ruby 的类变量继承机制复杂，子类修改类变量可能会影响父类或其他子类。
- **推荐使用类实例变量代替类变量**：
  - 类实例变量（如 `@class_instance_var`）结合类方法和 `self` 使用，更安全且直观。

### 类实例变量示例：
```ruby
class Person
  @population = 0

  def self.population
    @population
  end

  def self.increment_population
    @population += 1
  end

  def initialize(name, age)
    @name = name
    @age = age
    self.class.increment_population
  end
end

person1 = Person.new("Alice", 30)
person2 = Person.new("Bob", 25)

puts Person.population # 2
```

这样更符合面向对象编程的封装原则。