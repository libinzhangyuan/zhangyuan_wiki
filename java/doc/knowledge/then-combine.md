[返回](/java/doc/knowledge/index)


```
组合
组合多个Future
最常用的方法之一是thenCombine。这个方法允许你组合两个独立的CompletableFuture，
并且当它们都完成时，可以对它们的结果进行一些操作。

来看个例子：

CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
    simulateTask("加载用户信息");
    return "用户小黑";
});

CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
    simulateTask("加载订单数据");
    return "订单123";
});

CompletableFuture<String> combinedFuture = future1.thenCombine(future2, (userInfo, orderInfo) -> {
    return "合并结果：" + userInfo + "，" + orderInfo;
});

combinedFuture.thenAccept(System.out::println);
在这个例子中，future1和future2代表两个独立的异步操作。
只有当两者都完成时，thenCombine里面的函数才会执行，并且合并它们的结果。




```