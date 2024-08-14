[返回](/java/doc/knowledge/index)


```
任何一个future执行完就触发

        CompletableFuture<String> fast = fetchFast(); 
       
 
        CompletableFuture<String> predictable = fetchPredictably(); 
       
 
        fast.acceptEither(predictable, s -> { 
       
 
             
        System.out.println( 
        "Result: " 
         + s); 
       
 
        }); 
       

```