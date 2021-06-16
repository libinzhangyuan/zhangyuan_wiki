* [异步编程](https://www.dartcn.com/guides/libraries/library-tour#dartasync---%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B)
* [Asynchrony Support: Phase 1](https://www.dartcn.com/articles/language/await-async)
* [Asynchrony Support: Phase 2](https://www.dartcn.com/articles/language/beyond-async)

```
声明异步函数 Future<String> lookUpVersion() async => '1.0.0';
等待异步函数返回：await

同步生成器 Synchronous 生成器： 返回一个 Iterable 对象。 函数内标记 sync*
异步生成器 Asynchronous 生成器： 返回一个 Stream 对象。  函数内标记 async*
如果生成器是递归的，可以使用 yield* 来提高其性能：


Isolates







```