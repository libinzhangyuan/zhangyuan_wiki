[返回](/java/doc/knowledge/index)


```

3. 组合异步操作时的错误处理
当你组合多个CompletableFuture时，记得对每一个Future都进行错误处理。这样可以避免一个未捕获的异常破坏整个操作链。

CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "任务1").exceptionally(ex -> "默认值1");
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "任务2").exceptionally(ex -> "默认值2");

CompletableFuture<String> combinedFuture = future1.
    thenCombine(future2, (result1, result2) -> result1 + " 和 " + result2);
这样做确保了即使其中一个操作失败，整个链也可以继续执行。



```