```
可以在这个链接里面搜索案例
https://openresty.org/download/agentzh-nginx-tutorials-zhcn.html



set $foo hello;
echo "foo: $foo";

转义$符号:
geo $dollar {
        default "$";
    }
location {echo "dollor sign: $dollar"}


内部跳转 echo_exec


```