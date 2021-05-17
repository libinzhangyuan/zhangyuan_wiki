```
??  ??=  避空运算符 a ??= 3   # ruby中，  a = 3 if a.nil?

条件属性访问  a?.member    等效 (a != null) ? a.member : null   # ruby 中的 try 语法

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

```