```
创建出来后，允许test时，报错
Failed to load ApplicationContext for xxxxxx
https://www.cnblogs.com/zwb-19981125/p/13213008.html
加一个自动配置注解
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})



```