[返回](/java/doc/knowledge/index)

### 依赖关系的处理thenCompose
```

如果你的一个异步操作依赖于另一个异步操作的结果，那么可以使用thenCompose方法。
这个方法允许你在一个Future完成后，以其结果为基础启动另一个异步操作。

CompletableFuture<String> masterFuture = CompletableFuture.supplyAsync(() -> {
    simulateTask("获取主数据");
    return "主数据结果";
});

CompletableFuture<String> dependentFuture = masterFuture.thenCompose(result -> {
    return CompletableFuture.supplyAsync(() -> {
        simulateTask("处理依赖于" + result + "的数据");
        return "处理后的数据";
    });
});

dependentFuture.thenAccept(System.out::println);
这个例子中，dependentFuture的执行依赖于masterFuture的结果。


```