当然，以下是用 Markdown 格式写的解释：

```markdown
## `append_features` 和 `included` 的区别

在 Ruby 中，`append_features` 和 `included` 都是与模块（`module`）被包含到类（`class`）
或其他模块中时相关的回调方法。它们的区别主要体现在调用时机和用途上。

### 1. `append_features`

- **作用**：`append_features` 是一个模块方法，当模块被包含到类或模块中时会被调用。
它的默认行为是将模块的实例方法添加到目标类或模块的继承链中。
- **用法**：你可以重写 `append_features` 来实现一些自定义的逻辑，比如修改目标类或模块的行为、记录日志等。
- **调用时机**：当模块通过 `include` 被包含到类或模块时，`append_features` 会在此时调用。

```ruby
module MyModule
  def self.append_features(base)
    super
    puts "#{base} included MyModule"
  end
end

class MyClass
  include MyModule
end
```

输出：
```
MyClass included MyModule
```

### 2. `included`

- **作用**：`included` 是一个模块的回调方法，它在模块被包含到类或模块时被触发。
`included` 方法通常用于执行一些初始化的操作，比如设置类的属性，或是通知其他模块。
- **用法**：`included` 主要用于做一些初始化的工作，通常与 `extend` 结合使用，或者在类或模块中动态添加方法。
- **调用时机**：`included` 会在模块被 `include` 到类或模块之后立即被调用。

```ruby
module MyModule
  def self.included(base)
    puts "#{base} included MyModule"
  end
end

class MyClass
  include MyModule
end
```

输出：
```
MyClass included MyModule
```

### 总结

- `append_features` 主要用于向类添加模块的实例方法，它的调用时机是在模块被包含时。
- `included` 也是在模块被包含时触发，但是它更常用于在类或模块中做初始化操作。

在实际使用中，`included` 更常见，因为它提供了一个更加直观的方式来处理模块包含后的操作。
```

这样你就能在 Markdown 环境中清晰地展示 `append_features` 和 `included` 的差别了！