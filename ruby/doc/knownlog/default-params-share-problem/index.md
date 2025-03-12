

```
默认参数的注意事项

    默认参数的位置：默认参数通常放在参数列表的末尾，以避免在调用方法时产生歧义。
    默认值的计算：默认值在方法定义时计算，而不是在方法调用时计算。因此，如果默认值是一个可变对象（如数组或哈希），它将在所有方法调用之间共享。

def add_item(item, list = [])
  list << item
  puts list.inspect
end

add_item("apple")  # 输出: ["apple"]
add_item("banana") # 输出: ["apple", "banana"]

在这个例子中，list 参数的默认值是一个空数组。由于默认值在方法定义时计算，所有调用 add_item 方法时都会共享同一个数组。

```