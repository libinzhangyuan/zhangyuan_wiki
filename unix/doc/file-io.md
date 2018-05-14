# File IO

### 文件数据结构模型 模型
```
三个相关数据结构：
文件描述符表(a table of open file descriptors)
-   存储 文件描述符id 和 文件描述符标志(close-on-exec)    set_fd
文件表
-   存储文件状态标志(O_RDONLY, O_APPEND, O_ASYNC等)，当前文件偏移量    set_fl    (set file status flag)
v节点表
-   存储节点信息和当前文件大小 等更多静态的信息。

fork和dup后，共用一个文件表项
进程调用fork后，子进程和父进程的文件描述符所对应的文件表项是共享的，
这意味着子进程对文件的读写直接影响父进程的文件位移量(反之同理)。
dup一样的原理。
https://www.tuicool.com/articles/6F3Ejy
```

### fflush和fsync的联系和区别
```
1.提供者fflush是libc.a中提供的方法，fsync是系统提供的系统调用。
2.原形fflush接受一个参数FILE *.fflush(FILE *);
fsync接受一个Int型的文件描述符。fsync(int fd);
3.功能fflush:是把C库中的缓冲调用write函数写到磁盘[其实是写到内核的缓冲区]。
     fsync：是把内核缓冲刷到磁盘上。 
c库缓冲-----fflush---------〉内核缓冲--------fsync-----〉磁盘
```

### 以append模式打开文件后。
```
每次写时都加到文件的尾端， 因此可以lseek来控制读取其他位置，但无法控制写其他位置。

```

### 文件/目录权限
```
目录的执行权限和读权限的作用：
    要ls一个目录，需要有此目录的读权限
    要open一个文件，需要有此文件所在目录和上层每一个目录的执行权限。 所以执行权限位也被叫做搜索位(search bit)
    要创建一个文件，需要有此目录的写权限

```
