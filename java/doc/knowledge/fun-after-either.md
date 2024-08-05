[返回](/java/doc/knowledge/index)


```

        CompletableFuture<String> fast = fetchFast(); 
       
 
        CompletableFuture<String> predictable = fetchPredictably(); 
       
 
        fast.acceptEither(predictable, s -> { 
       
 
             
        System.out.println( 
        "Result: " 
         + s); 
       
 
        }); 
       

```