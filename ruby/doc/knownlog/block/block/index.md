### 什么是block
```
# block
{puts x += 1}


函数使用上两种方法
def func
  yield
end

def func(&block)
 block.call
end

```


### 元编程
```


def hello
  判断本函数是否有代码块
  if block_given?
  end
end
```