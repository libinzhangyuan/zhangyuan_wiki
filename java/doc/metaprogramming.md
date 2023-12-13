```
从对象获取类对象：  final Class<?> getClass()      classObj = user.class
从类型获取类对象： classObj = User.class
Class的方法，变量，创建函数： Method m = getMethod();  getField() getConstructor()
获取注解：对Class,Method,Field,Constructor对象调用getAnnotation() 或 getAnnotations()
Annotation anno = m.getAnnotation()
Annotation annos[] = m.getAnnotations(); 


Class.forName("TestClass"); 


类型通配符 List<?> data
https://www.runoob.com/java/java-generics.html
类型通配符上限形如 List<? extends Number>
类型通配符下限形如 List<? super Number> 

```
