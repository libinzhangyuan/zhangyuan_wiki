# Maven命令

### 依赖查看
```
mvn dependency:resolve 查看依赖
mvn dependency:tree 查看完整的依赖路径
mvn install -X 更完整
```


### 运行
```
mvn exec:java 运行jar包。 
可以自动在.m2目录中，寻找到依赖的包。线下环境使用
cd simple-weather
mvn exec:java -Dexec.mainClass=org.sonatype.mavenbook.weather.Main

如果用java命令，则需要 -cp指定好依赖的包的完整路径。
cd simple-weather
java -cp ./target/simple-weather-1.0.jar:./target/log4j-1.2.14.jar org.sonatype.mavenbook.weather.Main


mvn test 运行测试
```


### 构建线上包
```
mvn install      mvn clean install
mvn assembly:assembly # 构建线上包

```

