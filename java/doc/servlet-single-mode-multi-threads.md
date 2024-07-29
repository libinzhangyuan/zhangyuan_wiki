[返回](/java/doc/multithread)


https://developer.aliyun.com/article/581565


### 但是注意，文档里面关于servletContext的线程安全描述是错误的, 改为如下
```
在多线程环境中，ServletContext 本身是线程安全的。这意味着你可以在多个线程中安全地访问同一个 ServletContext 实例，而不必担心数据竞争或不一致的问题。然而，这并不意味着你可以直接修改 ServletContext 中存储的数据而不需要同步。如果需要修改 ServletContext 中存储的属性或数据，你应该使用适当的同步机制来确保线程安全。

例如，如果你在 ServletContext 中存储了一个集合，并希望在多个线程中修改这个集合，你应该确保对这个集合的访问是同步的，或者使用线程安全的集合类，如 Collections.synchronizedList() 或 ConcurrentHashMap。

总的来说，ServletContext 提供了线程安全的访问，但如果你打算修改它所包含的数据，需要自己确保操作的线程安全性。

```


