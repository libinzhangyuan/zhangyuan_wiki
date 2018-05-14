类体系：
类体系图 https://www.cnblogs.com/zemliu/archive/2013/08/19/3269015.html
总概：http://blog.csdn.net/hguisu/article/details/7418161

### 字节流 字节输入-输出
```

#原始流处理器 接收Byte数组对象,String对象,FileDescriptor对象将其适配成InputStream,以供其他装饰器使用,
InputStream
->ByteArrayInputStream
->FileInputStream
->PipedInputStream
->StringBufferInputStream (已废弃)

链接流处理器 接收另一个流处理器(InputStream,包括链接流处理器和原始流处理器)作为源,并对其功能进行扩展,所以说他们是装饰器.
InputStream
->FilterInputStream
--->BufferedInputStream: 用来将数据读入内存缓冲区,并从此缓冲区提供数据
--->DataInputStream: 提供基于多字节的读取方法,可以读取原始数据类型(Byte, Int, Long, Double等等)
--->LineNumberInputStream:  提供具有行计数功能的流处理器
--->PushbackInputStream:  提供已读取字节"推回"输入流的功能

InputStream
->SequenceInputStream 同时读取多个流的装饰器

InputStream
->ObjectInputStream  implements ObjectInput, ObjectStreamConstants

RandomAccessFile implements DataOutput, DataInput
针对完全随机读写文件的场景。没有缓冲，因此
https://www.ibm.com/developerworks/cn/java/l-javaio/index.html



OutputStream
->FilterOutputStream
- ->PrintStream   System.out就是PrintStream. 
- ->DataOutputStream

```


### 字符流 字符输入-输出体系 处理unicode字符
