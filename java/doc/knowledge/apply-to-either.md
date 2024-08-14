[返回](/java/doc/knowledge/index)


```

时间优先转换–CompletableFuture.applyToEither()

CompletableFuture.applyToEither() 用于连接两个独立的CompletableFuture，
选择计算（返回结果）最快的一个CompletableFuture，进行转换计算操作(Function<? super T, U>)并返回结果。

<U> CompletionStage<U> applyToEither(CompletionStage<? extends T> other,Function<? super T, U> fn);
<U> CompletionStage<U> applyToEitherAsync(CompletionStage<? extends T> other,Function<? super T, U> fn);
<U> CompletionStage<U> applyToEitherAsync(CompletionStage<? extends T> other,Function<? super T, U> fn,Executor executor);

我们现实开发场景中，总会碰到有两种渠道完成同一个事情，所以就可以调用这个方法，找一个最快的结果进行处理。一个例子如下：

String result = CompletableFuture.supplyAsync(() -> {
         try {
             Thread.sleep(3000);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
         return "s1";
     }).applyToEither(CompletableFuture.supplyAsync(() -> {
         try {
             Thread.sleep(2000);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
         return "hello world";
     }), s -> s).join();  //2000毫秒后返回"hello world"


```