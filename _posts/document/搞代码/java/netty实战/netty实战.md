## Netty简介
| 分类     | Netty的特性                                     |
| -------- | ----------------------------------------------- |
| 设计     | 统一的API,支多中传输类型,则色和非阻塞的线程模型 |
| 易于使用 | 基于完全基于JDK不需要其他依赖                   |
| 性能     | 拥有高于java API 的更高吞吐量                   |
| 健壮性   | 不会因为慢速,快速导致OOM                        |
| 安全性   | 完成的SSL/TLS可用于受限环境                     |

## 第一章
本书第一章简单的介绍了Netty的构造以及代码结构和主要的工具类,实际上我们就是通过调用Netty的API来实现我们的NIO.

### Netty核心组件
- Channel;
- 回调;
- Future
- 事件和ChannelHandler.

#### Channel
Channel是Java NIO的一个基本构造.可以吧Channel看做传入(入栈)或者传出(出栈)的数据载体.

#### 回调
回调其实就是一个方法,一个执行已经提供给另一个方法的方法的引用.这使得后者可以在适当的时候调用前者.

在Netty中使用回调来处理时间;当一个回调被触发时相关的时间被一个interfaceChannelHandler的实现处理.当一个新的连接建立是ChannelHandlerchannelActive()回调将会被调用.

```
public class ConnectHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx)
        throws Exception {    ← --  当一个新的连接已经被建立时，channelActive(ChannelHandlerContext)将会被调用
        System.out.println(
            "Client " + ctx.channel().remoteAddress() + " connected");
    }
}
```

#### Future
Future提供了另一种在操作完成时通知应用程序的方式.可以看做是一个一步操作的结果的占位符;它将在未来某个时刻完成,并提供对其结果的访问.

#### 事件和ChannelHandler
Netty使用不同的事件来通知我们状态的改变或者是操作的状态.

入栈事件:
- 连接已被激活或者连接失活;
- 数据读取;
- 用户事件;
- 错误时间.

出栈事件:
- 打开或者关闭到远程节点的连接;
- 将数据写到或者冲刷到套接字.

![](http://ws1.sinaimg.cn/large/006tKfTcly1g1n938y959j30ka05gdga.jpg)

### 放到一起

#### Future、回调和ChannelHandler

#### 选择器、事件和EventLop

## 第二章
本书第二站带着我们创建了一个简单的Netty应用(包含一个server端和一个client端)通过两端的交互可以获得Netty的初步应用体验.

## 客户端/服务端概念
![](http://ws3.sinaimg.cn/large/006tKfTcly1g1n9b2tnldj30kk0g875l.jpg)

## 第三章
从更高层次的角度来看,Netty解决了两个相应的关注领域,可以将其大致的标记为技术的和体系结构的.
- 基于Java NIO的异步和时间驱动的实现,保证了高负载下应用程序性能的最大化;
- 包含一组设计模式,将应用逻辑从网络层解耦,简化了开发过程.

### Channel、EventLopp和ChannelFuture
将这些类和在一起,可以被人为是Netty网络抽象的代表:
- Channel-->Socket;
- EventLoop-->控制流、多线程处理、并发；
- ChannelFuture-->异步通知.

### Channel
基本的I/O操作(bind()、connect()、read()和write())Netty的Channel接口提所提供的API看,大大降低了直接使用Socket类的复杂性.Channel也拥有许多的预定义:
- EmbeddedChannel;
- LocalServerChannel;
- NioDatagramChaannel.

### EventLoop接口
EventLoop定义了Netty的核心抽象,用于处理连接的声明周期中所发生的事件.

![关系图](http://ws2.sinaimg.cn/large/006tKfTcly1g1n9lfhd0cj30kq0godi1.jpg)

根据上面的关系图可知:
- 一个EventLoopGroup包含一个或者多个EventLoop;
- 一个EventLoop在他的声明周期内只和一个Thread绑定;
- 所有由EventLoop处理的I/O事件都将在他专有的Thread上被处理;
- 一个Channel在他的声明周期内只注册一个EventLoop
- 一个EventLoop可能会被分配给一个或者多个Channel.

### ChannelFuture接口
Netty提供了ChannelFuture接口,其addListener()方法注册了一个ChannelFutureListener,一遍在扣个操作完成是(无论是否成功)得到通知, **所有同属于一个Channel的操作都会保证被顺序执行**.

## ChannelHandler和ChannelPipeline
详细的看一下管理数据流和处理应用程序逻辑的组件.

### ChannelHandler接口
从开发人员的角度来讲,Netty最主要的组件就是ChannelHndler,他充当了所有处理入栈和出栈数据的应用程序逻辑的容器.因为Channel的方法是有**网络事件**出发的.
<div style="color:red">在开发过程中我们的业务逻辑往往驻留在一个或者多个ChannelHandler中</div>
### ChannelPipeline接口
ChannelPipeline为ChannelHandler链提供了容器,并定义了传播入栈和出栈事件流的API,当Channel被创建时,他会被自动地分配到它专属的ChannelPipline.
![派生关系](http://ws4.sinaimg.cn/large/006tKfTcly1g1nav7v9dpj30kk067t95.jpg)
上图说明了ChannelHandler的两个实现一个是==in==一个是==out==,其在ChannelPipline中的作用也是不一样的.
![入栈出栈过程](http://ws4.sinaimg.cn/large/006tKfTcly1g1naxsmp0zj30km05pgm0.jpg)
如果一个消息或者任何入栈事件被读取,那么它会从ChannelPipeline的头部开始流动,并传递给第一个ChannelInboundHandler,最终会将消息传递到尾端.

数据出栈从概念上来将也是一样的.

## 引导

Netty的引导类为应用程序的网络层提供了容器,这设计讲一个进程绑定到某个指定的端口,或者讲一个进城连接到另一个运行在某个指定主机的指定端口上的进程.

"服务器"和"客户端"实际上表示了不同的网络行为;换句话说,时间听传入的丽娜姐还是建立一个或者多个进程的连接.

因此,有两种类型的引导:一种用于客户端(简称为Bootstrap),另一种(ServerVootstrap)用于服务器,无论使用哪种协议或者处理哪种类型的数据,唯一决定它使用哪种引导类的是他作为一个客户端还是一个服务器.

## 第四章
第四章主要讲解了Netty的概念以及传输的方式.

### 传输API
传输API的核心是interface Channel,他被用于所有的I/O操作.
![传输的核心](http://ws1.sinaimg.cn/large/006tKfTcly1g1pq20n6o9j30kh074gme.jpg)

如图所示,每一个Channel都包含一个ChannelPipeline和ChannelConfig,并且ChannelConfig中的配置可以支持热更新.

**ChannelPipeline**持有所有应用用于入栈和出栈数据以及事件的ChannelHandler实例

 #### ChannelHandler典型用途
 - 将数据从一种格式转换为另一种格式;
 - 提供异常的通知;
 - 提供Channel变为活动的或者非活动的通知;
 - 提供Channel注册或者注销的通知;
 - 提供用户自定义事件通知.

### NIO--非阻塞I/O
NIO提供了一个所有I/O操作的异步的实现.选择器的基本概念就是充当一个注册表,在哪里可以在请求Channel的状态发生变化的时候得到通知.
- 新的Channel已被接受并就绪;
- Channel连接已经完成;
- Channel有意境就绪的提供读取的数据;
- Channel可用于写数据

选择器运行在一个检查状态变化并对其作出相应相应的线程上,在应用程序对出状态的改变作出相应之后,选择器将会被重置,并将重复这个过程.

![选择器的处理流程](http://ws4.sinaimg.cn/large/006tKfTcly1g1pqp313ukj30kh0cndih.jpg)

#### Epoll--用于Linux的本地非阻塞传输
Netty的NIO传输基于Java提供的异步/非阻塞网络编程的通用抽象.这使得Netty的非阻塞API可以在任何平台上使用,但是也包含了相关的限制.

Epoll的网络模型与上图NIO的完全相同

#### 用于JVM内部通信的Local传输
Netty提供了一个Local传输,用于在同一个JVM中运行客户端和服务器程序之间的异步通信.
![内部通信流程图](http://ws4.sinaimg.cn/large/006tKfTcly1g1pqzmpp4oj30km0cxwgd.jpg)

#### Embedded传输
可以将一组ChannelHandler作为帮助其类其纳入到其他的ChannelHandler内部.

### 传输用例
并不是所有的传输都支持所有的协议的
![传输协议](http://ws1.sinaimg.cn/large/006tKfTcly1g1pr2r6yghj30bl04gwei.jpg)

## 第五章
- ByteBuf--Netty的数据容器
- API的详细信息
- 用例
- 内存分配

### ByteBuf的API
列出ByteBuf的优点:
- 被用户自定义的缓冲区类扩展;
- 通过内置的符合缓冲区类型实现了零拷贝
- 容量可以按需增长
- 在读和写之间方便切换
- 读和写使用了不同的索引
- 方法支持链式调用
- 支持引用计数
- 支持池化

### ByteBuf类--Netty的数据容器

## 第六章 ChannelHandler和ChannelPipeline

