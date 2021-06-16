```
??  ??=  避空运算符 a ??= 3   # ruby中，  a = 3 if a.nil?

条件属性访问  a?.member    等效 (a != null) ? a.member : null   # ruby 中的 try 语法


获取对象的类型: a.runtimeType
as、is、is! 运算符是在运行时判断对象类型的运算符。 https://dart.cn/guides/language/language-tour#type-test-operators

箭头语法 (便捷的定义短的函数) => aListOfStrings.any((s) => s.isEmpty);

级联语法 ..   myObject..someMethod()
querySelector('#confirm')
..text = 'Confirm'
..classes.add('important')
..onClick.listen((e) => window.alert('Confirmed!'));

类中getter setter写法   int get aProperty => _aProperty;
set aProperty(int value) {}

函数定义可选位置参数 int func(int a, [int b = 3, int c])
函数定义可选命名参数 int func(int a, {String b})    调用 func(1, b: 's')

构造函数中使用this:   MyColor(this.red, {this.green, this.blue = 80})   直接给成员变量赋值

命名构造方法(构造函数) and 构造函数初始化列表
                  Point.fromJson(Map<String, num> json)
                    : x = json['x'],
                      assert(json['b'] > 3) {}

工厂构造方法：  factory Shape.fromJson(Map<String, num> json) {}

重定向构造函数:   Automobile.hybrid(String make ,String model) : this(make, model, 60);

const构造方法  const ImmutablePoint(this.x)

判断两个变量是否其实是同一个变量  assert(identical(a, b))


定义变量可以为null    double? x;
定义成员变量初始值为null         class XXX { double? x }; // Declare instance variable x, initially null.

相等比较 == 比较的hash指   identical 比较的是否引用的同一个对象


```

* [变量](https://www.dartcn.com/guides/language/language-tour#%E5%8F%98%E9%87%8F)
* [covariant keyword 协变量](https://www.dartcn.com/guides/language/sound-problems#the-covariant-keyword) 比如需要父类参数，但传给它子类对象时，使用covariant来表示不要报错，代码故意的
* [多线程,异步,stream](async)