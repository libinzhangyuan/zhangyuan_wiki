* [proc](proc/index)
* [lambda](lambda/index)

* [proc和lambda的区别](proc-lambda-diff/index)

* [block](block/index)

* []


```
def foo(&block)
  a = 3
  block.call(a)
end

这个& 是将block转换为proc的语法
```

```

arr.each &:to_s
这个&是将proc转换为block的语法。
啊秋的视频中说 method会自动转换为proc

```