[返回](/java/doc/multithread)


https://developer.aliyun.com/article/581565


### 但是注意，文档里面关于servletContext的线程安全描述是错误的, 改为如下
```
在多线程环境中，ServletContext 本身是线程安全的。
这意味着你可以在多个线程中安全地访问同一个 ServletContext 实例，而不必担心数据竞争或不一致的问题。
然而，这并不意味着你可以直接修改 ServletContext 中存储的数据而不需要同步。
如果需要修改 ServletContext 中存储的属性或数据，你应该使用适当的同步机制来确保线程安全。

例如，如果你在 ServletContext 中存储了一个集合，并希望在多个线程中修改这个集合，
你应该确保对这个集合的访问是同步的，或者使用线程安全的集合类，
如 Collections.synchronizedList() 或 ConcurrentHashMap。

总的来说，ServletContext 提供了线程安全的访问，
但如果你打算修改它所包含的数据 比如其中的map,array等，需要自己确保 "操作包含的数据结构时" 的线程安全性。

```


### 但是注意，文档里面关于HttpSession的线程安全描述是错误的, 改为如下
```
HttpSession 对象本身是线程安全的，因为Servlet容器确保了对每个会话的访问是由一个线程来处理的。这意味着在一个HTTP请求的生命周期内，一般情况下只有一个线程可以访问和修改HttpSession对象。
特别找情况 就是浏览器开多个窗口，多个窗口的请求恰好在同一时间到达. 需要处理么?

```


### 但是注意，文档里面关于ServletRequest的线程安全描述是错误的, 改为如下
```
Servlet的ServletRequest对象不是线程安全的。在Java Servlet API中，ServletRequest对象代表了一个客户端发出的请求，它包含了请求的信息，比如参数、头信息等。每个请求都是独立处理的，因此ServletRequest对象通常只在处理特定请求的线程中使用。

由于ServletRequest对象是请求特定的，它不会在多个线程之间共享，所以它本身不需要是线程安全的。但是，如果你在ServletRequest对象上存储了可变的数据，并且这个数据可能会被多个线程访问，那么你需要确保对这个数据的访问是线程安全的。

在Servlet中，通常使用单例模式来实现线程安全的服务端资源，而请求相关的数据和操作则在请求的生命周期内由单个线程处理，从而避免了线程安全问题。


```