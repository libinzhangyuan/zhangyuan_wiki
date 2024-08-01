[返回](/java/doc/knowledge/index)

```
thenApply用于处理和转换CompletableFuture的结果。
thenAccept用于消费CompletableFuture的结果，不返回新的CompletableFuture。
thenRun则不关心前一个任务的结果，只是在前一个任务执行完后，执行一些后续操作。


CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    simulateTask("查询数据库");
    return "查询结果";
});

future.thenApply(result -> {
    // 对结果进行处理
    return "处理后的结果：" + result;
}).thenAccept(processedResult -> {
    // 消费处理后的结果
    System.out.println("最终结果：" + processedResult);
}).thenRun(() -> {
    // 执行一些不需要前一个结果的操作
    System.out.println("所有操作完成");
});

在这个例子里，小黑用supplyAsync启动了一个异步任务来查询数据库。
然后用thenApply处理查询结果，用thenAccept消费处理后的结果，最后用thenRun标记所有操作完成。

```