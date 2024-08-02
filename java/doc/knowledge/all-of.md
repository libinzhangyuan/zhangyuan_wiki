[返回](/java/doc/knowledge/index)

```

处理多个Future
有时候，咱们可能有多个异步操作，需要等所有操作都完成后再进行下一步。这时候，可以使用CompletableFuture.allOf。

CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
    simulateTask("任务一");
    return "结果一";
});

CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
    simulateTask("任务二");
    return "结果二";
});

CompletableFuture<Void> allFutures = CompletableFuture.allOf(future1, future2);

allFutures.thenRun(() -> {
    System.out.println("所有任务完成");
    future1.get();
    future2.get();
});
allOf会等待所有提供的Futures完成，然后执行后续操作。


```