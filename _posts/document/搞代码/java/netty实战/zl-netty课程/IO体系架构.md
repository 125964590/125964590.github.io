# IO体系架构

在java中整体分为字节流和字符流(字符流是为了方便操作包装了字节流)

![image-20191023105948950](https://tva1.sinaimg.cn/large/006y8mN6ly1g87yiukpb9j31lg0o6tf6.jpg)

![image-20191023110117923](https://tva1.sinaimg.cn/large/006y8mN6ly1g87yisub5fj317j0u0wpl.jpg)

![image-20191023110126409](https://tva1.sinaimg.cn/large/006y8mN6ly1g87yiozi4dj31an0u011y.jpg)

![image-20191023110217489](https://tva1.sinaimg.cn/large/006y8mN6ly1g87yinewhvj317v0u0duv.jpg)

![image-20191023110435296](https://tva1.sinaimg.cn/large/006y8mN6ly1g87yikakd9j31a40u0ken.jpg)

![image-20191023110556970](https://tva1.sinaimg.cn/large/006y8mN6ly1g87yij1qldj31ay0u0b29.jpg)

![image-20191023110802913](https://tva1.sinaimg.cn/large/006y8mN6ly1g87yifxsn6j314y0u0tqd.jpg)

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g87yjkw2dyj31ba0u0kc2.jpg)

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g87yp3xio8j31820setyb.jpg)

## Io 与nio

java.io中最核心的一个概念就是(Stream),面向流的编程.java中,一个流要么是输入流,要么是输出流,不可能同时既是输入流又是输出流

java.nio中拥有三个核心概念:Selector,Channel与Buffer.在java.nio中,我们是面向块(block)或是缓冲区(buffer)编程的.

Buffer本身就是一块内存,底层实现上,它实际上是一个数组.数据的读/写都是通过Buffer来实现的.

在读写之前调用flip方法

Java中的8中原画师呢个数据类型都有各自对应的Buffer类型,如IntBuffer,LongBuffer,ByteBuffer以及CharBuffer等等.

Channel指的是可以想起写入数据或者是从中读取数据的对象,它类似于java.io中的Stream.

所有数据的读写都是通过Buffer来进行的,永远不会出现直接向Channel写入数据的情况,或是直接从Channel读取数据的情况.

与Stream不同的是,Channel是双向的,一个流只可能是InputStream或是OutputStream,Channel打开后则可以进行读取,写入或是读写.

由于Channel是双向的,因此他能更好的反映出底层操作的系统的真是情况;在Linux系统中,底层操作系统的通道就是双向的.



