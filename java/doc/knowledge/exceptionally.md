[返回](/java/doc/knowledge/index)


```
异常处理
基本异常处理
在CompletableFuture的世界里，如果异步操作失败了，异常会被捕获并存储在Future对象中。
咱们可以使用exceptionally方法来处理这些异常。
这个方法会返回一个新的CompletableFuture，它会在原来的Future抛出异常时执行。

来看个例子：

CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    if (new Random().nextBoolean()) {
        throw new RuntimeException("出错啦!");
    }
    return "正常结果";
}).exceptionally(ex -> {
    return "错误的回退结果：" + ex.getMessage();
});

future.thenAccept(System.out::println);
这里，小黑创建了一个可能会失败的异步操作。如果抛出异常，exceptionally方法就会被调用，返回一个包含错误信息的回退结果。

细粒度的异常处理
有时候，咱们可能需要更细粒度的控制，比如只处理特定类型的异常，或者在异常发生时还想继续其他操作。
这时候，可以用handle方法。它可以同时处理正常的结果和异常情况。

CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    if (new Random().nextBoolean()) {
        throw new RuntimeException("出错啦!");
    }
    return "正常结果";
}).handle((result, ex) -> {
    if (ex != null) {
        return "处理异常：" + ex.getMessage();
    }
    return "处理结果：" + result;
});

future.thenAccept(System.out::println);
在这个例子中，无论异步操作是成功还是失败，handle方法都会被调用。如果有异常，它会处理异常；如果没有，就处理正常结果。

管道式异常处理
CompletableFuture还允许咱们创建一个异常处理的“管道”，这样就可以把多个异步操作链接起来，并在链的任意位置处理异常。

CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // 第一个异步操作
    return "第一步结果";
}).thenApply(result -> {
    // 第二个异步操作，可能会出错
    throw new RuntimeException("第二步出错啦!");
}).exceptionally(ex -> {
    // 处理异常
    return "在第二步捕获异常：" + ex.getMessage();
}).thenApply(result -> {
    // 第三个异步操作
    return "第三步使用结果：" + result;
});

future.thenAccept(System.out::println);
在这个例子中，小黑创建了一个包含三个步骤的异步操作链。如果第二步出错，异常会被捕获并处理，然后处理结果被传递到第三步。



```