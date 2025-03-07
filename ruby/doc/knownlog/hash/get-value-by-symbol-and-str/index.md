在 Ruby 中，如果你想让 Hash 的键既可以用 `Symbol` 也可以用 `String` 来获取对应的值，可以通过以下几种方式来实现：

### 1. **Hash#with_indifferent_access (Rails)**
Rails 提供了 `HashWithIndifferentAccess`，可以让 Hash 同时支持 Symbol 和 String 来访问键：
```ruby
hash = { "name" => "张三", age: 25 }
hash = hash.with_indifferent_access

puts hash[:name]   # 张三
puts hash["name"]  # 张三
puts hash[:age]    # 25
puts hash["age"]   # 25
```
✅ 推荐在 Rails 项目中使用，最方便。

---

### 2. **手动转化键**
用 `transform_keys(&:to_sym)` 或 `transform_keys(&:to_s)` 来统一键：
```ruby
hash = { "name" => "张三", age: 25 }
hash = hash.transform_keys(&:to_sym)

puts hash[:name]   # 张三
puts hash["name"]  # nil (仅支持 Symbol)
```
---

### 3. **自定义 Hash 类**
自己封装一个 Hash 类，通过 `[]` 方法重写访问逻辑：
```ruby
class IndifferentHash < Hash
  def [](key)
    super(key.to_s) || super(key.to_sym)
  end
end

hash = IndifferentHash.new.merge("name" => "张三", age: 25)
puts hash[:name]   # 张三
puts hash["name"]  # 张三
puts hash[:age]    # 25
puts hash["age"]   # 25
```
---

### 4. **Hash#dig**
`dig` 方法也可以用来同时访问 Symbol 和 String：
```ruby
hash = { "name" => "张三", age: 25 }
puts hash.dig(:name)   # 张三
puts hash.dig("name")  # 张三
```
---

### ✅ **推荐优先级**
| 环境                | 方案                  | 说明              |
|------------------|-------------------|----------------|
| Rails           | `with_indifferent_access` | 最简洁 |
| 纯 Ruby        | 自定义类        | 灵活、可扩展 |
| 兼容性强       | `dig`          | 无需额外代码 |

你打算用在 Rails 里面还是纯 Ruby? 我可以帮你写个通用扩展 😎