```
    T get() throws InterruptedException, ExecutionException ，
永久阻塞，直到返回结果值，允许中断，计算过程中所有的异常会包裹为新的ExecutionException实例再抛出。


    T get(long timeout, TimeUnit unit)throws InterruptedException, ExecutionException, TimeoutException ，
添加超时时间设定，如果超时会抛出TimeoutException，
如果获取到结果则释放并返回，允许中断，计算过程中所有的异常会包裹为新的ExecutionException实例再抛出。


    T join() ，
永久阻塞，直到返回结果值，不允许中断，计算过程中所有的异常会直接抛出。


    T getNow(T valueIfAbsent) ，
如果当前的计算结果为null，马上返回valueIfAbsent，否则调用join()的逻辑。

```