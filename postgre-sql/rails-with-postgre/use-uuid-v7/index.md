[返回](/postgre-sql/rails-with-postgre/index)

# 在 Rails 中设置主键为 UUID v7 的方法

要在 Rails 中使用 PostgreSQL 并设置 UUID v7 作为主键，你需要进行以下步骤：

## 1. 启用 PostgreSQL 的 UUID 扩展

首先确保你的数据库支持 UUID，并启用 pgcrypto 扩展：

```ruby
class EnableUuid < ActiveRecord::Migration[7.0]
  def change
    enable_extension 'pgcrypto' unless extension_enabled?('pgcrypto')
  end
end
```

## 2. 创建使用 UUID v7 作为主键的迁移

对于新表，可以在迁移中这样设置：

```ruby
class CreateModelsWithUuidV7 < ActiveRecord::Migration[7.0]
  def change
    create_table :your_table_name, id: :uuid, default: -> { "gen_random_uuid()" } do |t|
      # 其他字段...
      t.timestamps
    end
  end
end
```

## 3. 对于现有表添加 UUID v7 主键

如果你需要修改现有表：

```ruby
class ChangeIdToUuidV7 < ActiveRecord::Migration[7.0]
  def change
    add_column :your_table_name, :uuid, :uuid, default: -> { "gen_random_uuid()" }, null: false
    change_table :your_table_name do |t|
      t.remove :id
      t.rename :uuid, :id
    end
    execute "ALTER TABLE your_table_name ADD PRIMARY KEY (id);"
  end
end
```

## 4. 在模型中设置

在模型文件中不需要特别设置，Rails 会自动识别 UUID 主键。

## 注意事项

1. `gen_random_uuid()` 生成的是 UUID v4，PostgreSQL 原生不支持 v7。如果需要严格的 UUID v7，你有几个选择：
   - 使用 Ruby 生成 UUID v7 并作为默认值
   - 创建 PostgreSQL 函数来生成 UUID v7

2. 如果确实需要 UUID v7，可以考虑使用 `uuid7` gem 或其他 Ruby UUID 生成库：

```ruby
# 在 Gemfile 中添加
gem 'uuid7'
```

然后在迁移中使用：

```ruby
create_table :your_table_name, id: :uuid, default: -> { "'#{UUID7.generate}'" } do |t|
  # ...
end
```

3. 关联关系也需要相应调整，使用 `uuid` 类型作为外键。

4. 确保你的 Rails 版本支持 UUID 主键（Rails 5+ 都支持）。