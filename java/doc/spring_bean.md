### spring中xml配置
```
在Spring中，可以使用 init-method 和 destroy-method 在bean xml配置文件属性用于在bean初始化和销毁某些动作时。
这是用来替代 InitializingBean和DisposableBean接口
http://www.yiibai.com/spring/spring-init-method-and-destroy-method-example.html

用p namespace给bean传直接参数    p:username="root"
用p namespace给bean传其他bean   p:dataSource-ref="datasource"


load-on-startup
http://www.blogjava.net/xzclog/archive/2011/09/29/359789.html
标记容器是否在启动的时候就加载这个servlet(实例化并调用其init()方法)。
它的值必须是一个整数，表示servlet应该被载入的顺序. 越小优先级越高




```




### Java代码中的配置
```

Spring常用注解说明：
@Controller,    @Service,    @repository,    @Component        这4个注解都是将类标识为Bean
controller层    service层    dao层           将类标注为bean的通用注解，少用
https://www.cnblogs.com/guoziyi/p/6122471.html


@Controller @RequestMapping
@Autowired @Resource
https://www.cnblogs.com/leskang/p/5445698.html


@restcontroller与@controller的区别
https://www.cnblogs.com/lannoy/p/5976402.html
@Controller：修饰class，用来创建处理http请求的对象
@RestController：Spring4之后加入的注解，原来在@Controller中返回json需要@ResponseBody来配合，如果直接用@RestController替代@Controller就不需要再配置@ResponseBody，默认返回json格式。
https://www.tuicool.com/articles/vyq2muz


@RequestBody, @ResponseBody
http://blog.csdn.net/kobejayandy/article/details/12690555

@XmlRootElement   可配合@RestController返回xml
https://www.tuicool.com/articles/vyq2muz

@ControllerAdvice
https://www.tuicool.com/articles/EvuQVn


```
