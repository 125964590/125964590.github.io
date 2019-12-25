# Netty防止意外退出

在我们初学Netty的时候都是将Netty放在main函数中进行启动,由于main函数是一个主线他和我们工作中所遇到的容器(SpringBoot/Tomecat)是不一样的,后者通常都会帮助我们做一些事情,在这里讲一下如何优雅的防止Netty意外退出.

## Netty意外退出的原因

在Java运行的时候回存在一个主线程,和其他线程,还有守护线程,守护线程是不会单独存在的.

Netty本质上是一个线程组当main线程已经结束的时候Netty的NioEventLoop还是处于运行状态所以JVM进程不会退出.

NioEventLoop包含以下几点特性:

1. NioEventLoop是非守护线程;
2. NioEventLoop运行之后,不会主动退出;
3. 只有调用shutdown方法,NioEventLoop才回退出.

一起看一下下面这段代码,其中有两处调用同步阻塞,其目的就是为了防止线程未启动就执行到了finally中的关闭方法

```java
    try {
      ServerBootstrap serverBootstrap = new ServerBootstrap();
      serverBootstrap
          .group(bossGroup, workerGroup)
          .channel(NioServerSocketChannel.class)
          .childHandler(new MyServerInitizlize());
      //调用同步阻塞
      ChannelFuture sync = serverBootstrap.bind(8899).sync();
      //调用同步阻塞
      sync.channel().closeFuture().sync();
    } finally {
      bossGroup.shutdownGracefully();
      workerGroup.shutdownGracefully();
    }
```

## 实际项目中的优化

在实际工作中我们不能通过同步阻塞的方法去实现,因为一般情况下在生产中使用Netty都是构建在其他三方的框架中所以直接通过注册监听器然后在监听器中释放资源就可以了:

- 初始化Netty服务端;
- 绑定监听端口;
- 想CloseFuture注册监听器,在监听器中释放资源;
- 调用方线程返回.