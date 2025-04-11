[返回](/ruby/doc/knownlog/index)

# Ruby 中 `dup` 和 `clone` 的区别

在 Ruby 中，`dup` 和 `clone` 都是用于创建对象的浅拷贝（shallow copy）的方法，但它们有一些重要的区别：

## 主要区别

1. **冻结状态(freeze)的复制**：
   - `clone` 会保留原对象的冻结状态（如果原对象被冻结，克隆的对象也会被冻结）
   - `dup` 不会复制冻结状态，新对象总是未冻结的

2. **单例方法(singleton methods)的复制**：
   - `clone` 会复制原对象的单例方法
   - `dup` 不会复制单例方法

3. **内部状态（如`taint`、`untrust`）的复制**：
   - `clone` 会复制这些内部状态
   - `dup` 也会复制这些状态（与冻结状态不同）

## 示例代码

```ruby
# 冻结状态示例
original = "original".freeze
cloned = original.clone
duped = original.dup

puts cloned.frozen?  # => true
puts duped.frozen?   # => false

# 单例方法示例
original = "original"
def original.special_method
  "special"
end

cloned = original.clone
duped = original.dup

puts cloned.respond_to?(:special_method)  # => true
puts duped.respond_to?(:special_method)   # => false
```

## 何时使用哪个

- 如果需要完全相同的副本（包括冻结状态和单例方法），使用 `clone`
- 如果只需要数据副本而不关心冻结状态或单例方法，使用 `dup`
- 对于大多数日常使用场景，`dup` 通常就足够了

## 注意事项

两者都是浅拷贝，对于包含引用类型属性的对象，嵌套对象不会被复制。如果需要深拷贝，需要自己实现或使用其他库。