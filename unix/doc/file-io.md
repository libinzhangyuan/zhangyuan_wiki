# File IO

### fflush和fsync的联系和区别
```
提供者fflush是libc.a中提供的方法，fsync是系统提供的系统调用。2.原形fflush接受一个参数FILE *.fflush(FILE *);fsync接受的时一个Int型的文件描述符。fsync(int fd);3.功能fflush:是把C库中的缓冲调用write函数写到磁盘[其实是写到内核的缓冲区]。fsync：是把内核缓冲刷到磁盘上。 
c库缓冲-----fflush---------〉内核缓冲--------fsync-----〉磁盘
```
