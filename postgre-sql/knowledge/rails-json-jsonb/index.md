[返回](/postgre-sql/knowledge/index)

# Rails 中使用 PostgreSQL 的 JSON 和 JSONB 类型

下面我将详细介绍在 Rails 应用中如何使用 PostgreSQL 的 JSON 和 JSONB 类型，包含丰富的 Rails 示例代码。

## JSON 与 JSONB 的区别

```
| 特性        | JSON 类型 | JSONB 类型 |
|------------|----------|-----------|
| 存储格式    | 文本格式  | 二进制格式 |
| 写入速度    | 更快     | 稍慢      |
| 查询速度    | 较慢     | 更快      |
| 支持索引    | 不支持   | 支持      |
| 保留空格    | 是       | 否        |
| 保留键顺序  | 是       | 否        |
| 保留重复键  | 是       | 否        |
```

## 基础用法

### 1. 创建迁移

```ruby
# 创建包含 JSON 和 JSONB 字段的表
class CreateProducts < ActiveRecord::Migration[6.1]
  def change
    create_table :products do |t|
      t.string :name
      t.json :specifications  # JSON 类型
      t.jsonb :metadata       # JSONB 类型
      t.timestamps
    end

    # 为 JSONB 字段添加 GIN 索引提高查询性能
    add_index :products, :metadata, using: :gin
  end
end
```

### 2. 模型中使用 JSON 字段

```ruby
class Product < ApplicationRecord
  # 设置默认值
  attribute :metadata, :jsonb, default: -> { { tags: [], ratings: [] } }
  
  # 验证 JSON 字段结构
  validate :validate_metadata_structure

  private

  def validate_metadata_structure
    unless metadata.is_a?(Hash)
      errors.add(:metadata, "必须是 Hash 类型")
    end
  end
end
```

## 增删改查操作示例

### 1. 创建记录

```ruby
# 创建新产品
product = Product.create(
  name: "MacBook Pro",
  specifications: { "color" => "space gray", "weight" => "1.4kg" },
  metadata: {
    "tags" => ["laptop", "apple"],
    "specs" => { "cpu" => "M1", "ram" => "16GB" },
    "price_history" => [1299, 1199, 1099]
  }
)

# 使用默认值创建
product = Product.create(name: "iPad", specifications: { "color" => "silver" })
puts product.metadata
# => {"tags"=>[], "ratings"=>[]}
```

### 2. 查询记录

```ruby
# 基本查询
black_products = Product.where("specifications->>'color' = ?", "black")

# 查询 JSONB 字段中包含特定键值
electronics = Product.where("metadata @> ?", { tags: ["electronics"] }.to_json)

# 查询嵌套属性
m1_chip_products = Product.where("metadata->'specs'->>'cpu' = ?", "M1")

# 查询数组包含特定元素
laptop_products = Product.where("metadata->'tags' ? :tag", tag: "laptop")
```

### 3. 更新记录

```ruby
# 更新整个 JSON 字段
product.update(metadata: { new_key: "new_value" })

# 更新部分字段 (Rails 6.1+)
product.metadata["specs"]["ram"] = "32GB"
product.save

# 使用 merge 更新部分字段 (不会覆盖其他字段)
product.update(metadata: product.metadata.merge({ "discount" => true }))

# 向数组添加元素
product.metadata["tags"] << "premium"
product.save
```

### 4. 删除记录中的 JSON 属性

```ruby
# 删除 JSON 字段中的某个键
product.metadata.delete("discount")
product.save

# 使用 SQL 删除 JSONB 字段中的键
Product.where(id: product.id).update_all("metadata = metadata - 'discount'")
```

## 高级用法

### 1. 添加作用域

```ruby
class Product < ApplicationRecord
  # 查找特定颜色的产品
  scope :with_color, ->(color) { where("specifications->>'color' = ?", color) }
  
  # 查找有特定标签的产品
  scope :with_tag, ->(tag) { where("metadata->'tags' ? :tag", tag: tag) }
  
  # 查找价格低于某值的产品
  scope :below_price, ->(price) { where("(metadata->>'price')::float < ?", price) }
end

# 使用作用域
Product.with_color("black").with_tag("electronics")
```

### 2. 自定义序列化

```ruby
class Product < ApplicationRecord
  serialize :specifications, ProductSpecificationsSerializer

  class ProductSpecificationsSerializer
    def self.dump(hash)
      hash.to_json
    end

    def self.load(json)
      json.is_a?(String) ? JSON.parse(json) : json
    end
  end
end
```

### 3. 使用 Store Accessor

```ruby
class Product < ApplicationRecord
  # 为 JSONB 字段创建虚拟属性
  store_accessor :metadata, :warranty_period, :manufacturer
  
  # 可以像普通属性一样使用
  validates :warranty_period, presence: true
end

# 使用 store_accessor
product = Product.new
product.warranty_period = "2 years"
product.manufacturer = "Apple"
product.save
```

### 4. 模型回调处理 JSON 数据

```ruby
class Product < ApplicationRecord
  before_save :normalize_metadata
  
  private
  
  def normalize_metadata
    # 确保 tags 是数组且元素是字符串
    metadata["tags"] = Array(metadata["tags"]).map(&:to_s)
    
    # 删除空值
    metadata.compact!
  end
end
```

## 性能优化

### 1. 添加索引

```ruby
class AddIndexesToProducts < ActiveRecord::Migration[6.1]
  def change
    # 为 JSONB 字段添加 GIN 索引
    add_index :products, :metadata, using: :gin
    
    # 为特定 JSONB 键添加索引
    add_index :products, "(metadata->>'warranty_period')", name: "index_products_on_warranty_period"
    
    # 为数组元素添加索引
    add_index :products, "(metadata->'tags')", using: :gin
  end
end
```

### 2. 使用原生 JSONB 操作提高性能

```ruby
# 批量更新 JSONB 字段的特定键
Product.where("metadata->>'manufacturer' = ?", "Apple")
       .update_all("metadata = jsonb_set(metadata, '{price}', '999')")

# 使用 || 操作符合并 JSONB 对象
Product.find(1).update("metadata = metadata || '{\"new_field\":\"value\"}'")
```

## 实际应用场景

### 1. 动态表单数据存储

```ruby
class SurveyResponse < ApplicationRecord
  # 使用 JSONB 存储动态表单响应
  store_accessor :response_data, :name, :email, :answers
end

# 使用示例
response = SurveyResponse.create(
  response_data: {
    name: "张三",
    email: "zhang@example.com",
    answers: {
      q1: "A",
      q2: ["B", "C"],
      q3: "自由文本回答..."
    }
  }
)
```

### 2. 产品多语言描述

```ruby
class Product < ApplicationRecord
  # 存储多语言描述
  attribute :descriptions, :jsonb, default: {}
end

# 使用示例
product = Product.create(
  name: "Smartphone",
  descriptions: {
    en: "High-end smartphone with advanced camera",
    zh: "高端智能手机，配备先进摄像头",
    ja: "高性能スマートフォン、先進的なカメラ搭載"
  }
)

# 获取当前语言描述
I18n.locale = :zh
product_desription = product.descriptions[I18n.locale.to_s]
```

### 3. 用户偏好设置

```ruby
class User < ApplicationRecord
  # 存储用户偏好设置
  attribute :preferences, :jsonb, default: {
    theme: "light",
    notifications: {
      email: true,
      push: false
    },
    language: "en"
  }
  
  def theme_dark?
    preferences["theme"] == "dark"
  end
end
```

## 最佳实践

1. **优先使用 JSONB**：除非有特殊需求，否则优先选择 JSONB 类型
2. **合理设计数据结构**：避免过度嵌套，保持数据结构相对扁平
3. **添加索引**：为经常查询的 JSONB 字段或键添加索引
4. **验证数据**：在模型中添加验证确保 JSON 数据结构符合预期
5. **考虑查询模式**：根据查询需求设计 JSON 结构，而不是存储需求
6. **文档化结构**：为团队记录 JSON 字段的预期结构

通过合理使用 PostgreSQL 的 JSON 和 JSONB 类型，可以在 Rails 应用中实现灵活的数据存储和高效查询。