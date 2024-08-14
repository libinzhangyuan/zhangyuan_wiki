[返回](/java/doc/knowledge/index)

```
时间优先消费–CompletableFuture.acceptEither()

CompletableFuture.acceptEither() 用于连接两个独立的CompletableFuture，
选择计算（返回结果）最快的一个CompletableFuture，进行消费操作(Consumer<? super T> action)，无返回值。

public CompletionStage<Void> acceptEither(CompletionStage<? extends T> other,Consumer<? super T> action);
public CompletionStage<Void> acceptEitherAsync(CompletionStage<? extends T> other,Consumer<? super T> action);
public CompletionStage<Void> acceptEitherAsync(CompletionStage<? extends T> other,Consumer<? super T> action,Executor executor);

一个例子如下：

CompletableFuture.supplyAsync(() -> {
         try {
             Thread.sleep(3000);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
         return "s1";
     }).acceptEither(CompletableFuture.supplyAsync(() -> {
         try {
             Thread.sleep(2000);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
         return "hello world";
     }), System.out::println); //2000毫秒后控制台打印 "hello world"

```