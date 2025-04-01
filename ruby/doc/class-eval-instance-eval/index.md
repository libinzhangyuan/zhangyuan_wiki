* [Ruby 中的 class_eval 和 instance_eval 全面介绍](instance-eval-for-class-and-instance/index)

* [这篇英文的讲的更好:Metaprogramming in Ruby: It's All About the Self](https://yehudakatz.com/2009/11/15/metaprogramming-in-ruby-its-all-about-the-self/)
<br>注意深刻理解这个：
instance_eval： breaks apart the self into the self that is used to execute methods and the self that is used when new methods are defined. When instance_eval is used, new methods are defined on the metaclass, but the self is the object itself.