[返回](/ruby/doc/knownlog/index)


[分享 Ruby 中 require,load,autoload,extend,include,prepend 的区别](https://ruby-china.org/topics/35350)



```

include
当一个类或者模块 include 了一个 module M 时，则该类或者模块就拥有了该 module M 的方法。


extend
当一个类使用 extend 时，相当于打开了该类的单件类，为其添加了单件方法。
当一个对象使用 extend时，相当于打开了该对象的单件类，为其添加了单件方法。


prepend 和 include 很像，当一个类 prepend 或 include 一个模块时，该模块中的方法会成为该类的实例方法。
二者的区别在于，模块在祖先链中的位置。使用 include 时，模块在包含它的类之上。如果是 prepend，则是在 prepend 它的类之下。而祖先链中位置的不同，决定了方法调用的顺序。



```