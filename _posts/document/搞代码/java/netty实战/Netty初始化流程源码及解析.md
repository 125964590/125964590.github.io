# Netty初始化流程源码解析

## 写在前面

Netty的初始化代码基本上都是一样的,但是Netty在底层到底做了什么事情呢?让我们跟着源代码来一探究竟吧~

## 初始化的模板代码

下面我把NettyServer端的代码贴出来,主要关注的是内部的启动流程所以外部的方法调用以及参数配置这里就通过注释简单声明,不做过多的介绍了.

```java
  public static void main(String[] args) throws InterruptedException {
    // 接受客户端的连接
    NioEventLoopGroup bossGroup = new NioEventLoopGroup();
    // 执行处理流程
    NioEventLoopGroup workerGroup = new NioEventLoopGroup();
    try {
      // 设定一些相关的参数
      ServerBootstrap serverBootstrap = new ServerBootstrap();
      // 队成员变量进行赋值
      serverBootstrap
          .group(bossGroup, workerGroup)
          // 在接受class对象时大多数都与反射有关
          .channel(NioServerSocketChannel.class)
          .childHandler(new MyServerInitizlize());
      // 重点关注的对象通过bind方法完成启动
      ChannelFuture sync = serverBootstrap.bind(8899).sync();
      // 同步阻塞等待channel关闭
      sync.channel().closeFuture().sync();
    } finally {
      bossGroup.shutdownGracefully();
      workerGroup.shutdownGracefully();
    }
  }
```

## Netty服务端创建时序图

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g8wkhgpggaj31dw0u0n3h.jpg)

通过这个时序图先来简单的了解一下:

1. 创建ServerBootstrap实例;
2. 设置并绑定Reactor线程池;
3. 设置并绑定服务端Channel();
4. TCP链路简历是创建ChannelPipeline();
5. 添加并设置ChannelHandler();
6. 绑定监听端口并启动服务端;
7. Selector轮询();
8. 网络时间通知;
9. 执行Netty系统和业务相关的Handler.

## NioEventLoopGroup

代码首先创建了两个线程组分别是bossGroup和workGroup,来自同样的对象,只是命名格式不同(为什么创建两个在下文进行详细讲解).

跟到构造函数内部可以发现类上的注释声明:这个类是用来处理Channel的.

跟随构造函数最终可以定位到父类的实现.

```java
        // 传入当前核数*2的线程数
				super(nThreads == 0 ? DEFAULT_EVENT_LOOP_THREADS : nThreads, executor, args);
				// 判断线程数设置是否非法
				if (nThreads <= 0) {
            throw new IllegalArgumentException(String.format("nThreads: %d (expected: > 0)", nThreads));
        }
				// 默认为空
        if (executor == null) {
            executor = new ThreadPerTaskExecutor(newDefaultThreadFactory());
        }
				// 开辟数组空间
        children = new EventExecutor[nThreads];
				......
        // 由于我们使用NioEventLoopGroup创建所以对应的实现就是NioEventLoop
        children[i] = newChild(executor, args);
				......
         for (EventExecutor e: children) {
            e.terminationFuture().addListener(terminationListener);
        }
				// 添加到集合中
        Set<EventExecutor> childrenSet = new LinkedHashSet<EventExecutor>(children.length);
        Collections.addAll(childrenSet, children);
        readonlyChildren = Collections.unmodifiableSet(childrenSet);
```

实际上整个构造函数没有做什么关键的事情,就是对一些数据进行初始化,这里要说明一下几点:

1. 创建的线程数是当前核数的2倍;
2. 根据线程数开辟了对应的Execute;
3. 声明了Selector的构建类型以及工厂,但没有真正的开启Selector;

## ServerBootstrap

Netty是一个Nio的通信框架,由于Java的网络编程代码比较复杂而且有很多固定配置方法,Netty为了统一多中网络编程的构建方式,为我们封装了一个ServerBootstrap的启动类.

上文的代码知识将这个启动类New出来了,没有在构造函数中设置任何参数是因为这个对象的参数太多了,通过构造函数设置反而非常麻烦.

Netty为了解决这个问题引入了构造器, 通过构造器将函数的参数一个个传递进去.

- **group**方法传递了两个参数,一个是**bossGroup**一个是**workGroup**,在其内部会将这个两个参数分别传递给AbstractBootstrap和ServerBootstrap.

- **channel**方法传入的是一个nioServerSocketChannel.class很显然这将会用到一个反射来创建对象,同样的道理这也是一个方法链调用,传入的参数赋值给了对象本身等带使用.

- **childHandler**方法传入的使我们自定义的Initizlize,这个Initizlize是交给childHandler使用的也就是workerGroup,同理ServerBootstrap也包含一个handler方法这个方法传入的同样也是一个channelHandler对象,这个对象是交给bossGroup使用的.

基本上ServerBootstrap方法的复制工作就完成了,接下来我们可以调用**bind()**方法来真正开启Netty

```java
      // 重点关注的对象通过bind方法完成启动
      ChannelFuture sync = serverBootstrap.bind(8899).sync();
```

上述的这个方法将服务绑定到8899的端口号并且开启同步等待.

>  这里需要提一下sync()这个方法,之所以要开启sync同步是因为bind()方法开启线程的逻辑没有执行完,整个main方法就结束了,具体可以查看我的另一篇文章[Netty初始化流程源码解析]()

跟到方法里面可以看到**doBind**方法

```java
 private ChannelFuture doBind(final SocketAddress localAddress) {
        final ChannelFuture regFuture = initAndRegister();
        final Channel channel = regFuture.channel();
        if (regFuture.cause() != null) {
            return regFuture;
        }

        if (regFuture.isDone()) {
            // At this point we know that the registration was complete and successful.
            ChannelPromise promise = channel.newPromise();
            doBind0(regFuture, channel, localAddress, promise);
            return promise;
        } else {
            // Registration future is almost always fulfilled already, but just in case it's not.
            final PendingRegistrationPromise promise = new PendingRegistrationPromise(channel);
            regFuture.addListener(new ChannelFutureListener() {
                @Override
                public void operationComplete(ChannelFuture future) throws Exception {
                    Throwable cause = future.cause();
                    if (cause != null) {
                        // Registration on the EventLoop failed so fail the ChannelPromise directly to not cause an
                        // IllegalStateException once we try to access the EventLoop of the Channel.
                        promise.setFailure(cause);
                    } else {
                        // Registration was successful, so set the correct executor to use.
                        // See https://github.com/netty/netty/issues/2586
                        promise.registered();

                        doBind0(regFuture, channel, localAddress, promise);
                    }
                }
            });
            return promise;
        }
    }
```

这段代码是对channel的初始化,并且返回一个ChannelFuture在初始化的过程中完成了一下几件事:

1. 通过反射创建channel对象(class信息是在ServerBootstrap的构造器中传入);
2. 设置channel的option和对应的attribute;
3. 异步添加abstract的handler;
4. 给child的handler添加对应的参数;
5. 异步将channel注册到group上;
6. 检测channel创建成功之后绑定端口.



## Future

继承自java的Future,表示存放一个计算的结果.

Netty的Future提供了更多的操作,这些操作扩展自Future,**主要扩展了Get方法**在使用的时候可以添加一个Listener当一个任务返程的时候立刻添加另一个任务去执行.

## ChannelFuture

在Netty中所有的IO操作都是异步的,在执行操作的时候当一个方法执行完成但是实际上并不一定完成这是会返回一个ChannelFuture

>  All I/O operations in Netty are asynchronous.  It means any I/O calls will
  return immediately with no guarantee that the requested I/O operation has
  been completed at the end of the call.  Instead, you will be returned with
  a {@link ChannelFuture} instance which gives you the information about the
  result or status of the I/O operation.

下图引用自ChannelFuture源码中的注释,表明了ChannelFuture

```java
                                       +---------------------------+
                                       | Completed successfully    |
                                       +---------------------------+
                                  +---->      isDone() = true      |
  +--------------------------+    |    |   isSuccess() = true      |
  |        Uncompleted       |    |    +===========================+
  +--------------------------+    |    | Completed with failure    |
  |      isDone() = false    |    |    +---------------------------+
  |   isSuccess() = false    |----+---->      isDone() = true      |
  | isCancelled() = false    |    |    |       cause() = non-null  |
  |       cause() = null     |    |    +===========================+
  +--------------------------+    |    | Completed by cancellation |
                                  |    +---------------------------+
                                  +---->      isDone() = true      |
                                       | isCancelled() = true      |
                                       +---------------------------+
```

提高了原有Future的可用性,提供了Listener去监听方法完成,

### 通过ChannelFactory(调用反射)创建Channel

```java
    final ChannelFuture initAndRegister() {
        Channel channel = null;
        try {
          //调用反射获取channel的实例
            channel = channelFactory.newChannel();
          //对channel进行赋值
            init(channel);
        } catch (Throwable t) {
            if (channel != null) {
                // channel can be null if newChannel crashed (eg SocketException("too many open files"))
                channel.unsafe().closeForcibly();
                // as the Channel is not registered yet we need to force the usage of the GlobalEventExecutor
                return new DefaultChannelPromise(channel, GlobalEventExecutor.INSTANCE).setFailure(t);
            }
            // as the Channel is not registered yet we need to force the usage of the GlobalEventExecutor
            return new DefaultChannelPromise(new FailedChannel(), GlobalEventExecutor.INSTANCE).setFailure(t);
        }

        ChannelFuture regFuture = config().group().register(channel);
        if (regFuture.cause() != null) {
            if (channel.isRegistered()) {
                channel.close();
            } else {
                channel.unsafe().closeForcibly();
            }
        }
```

进入init()方法

```java
    @Override
    void init(Channel channel) {
      //设置属性
        setChannelOptions(channel, options0().entrySet().toArray(newOptionArray(0)), logger);
        setAttributes(channel, attrs0().entrySet().toArray(newAttrArray(0)));

      //获取管道
        ChannelPipeline p = channel.pipeline();

        final EventLoopGroup currentChildGroup = childGroup;
        final ChannelHandler currentChildHandler = childHandler;
        final Entry<ChannelOption<?>, Object>[] currentChildOptions =
                childOptions.entrySet().toArray(newOptionArray(0));
        final Entry<AttributeKey<?>, Object>[] currentChildAttrs = childAttrs.entrySet().toArray(newAttrArray(0));

     //判断是否调用handler方法如果调用则添加到最后
        p.addLast(new ChannelInitializer<Channel>() {
            @Override
            public void initChannel(final Channel ch) {
                final ChannelPipeline pipeline = ch.pipeline();
                ChannelHandler handler = config.handler();
                if (handler != null) {
                    pipeline.addLast(handler);
                }

                ch.eventLoop().execute(new Runnable() {
                    @Override
                    public void run() {
                      //处理传入的handler初始化
                        pipeline.addLast(new ServerBootstrapAcceptor(
                                ch, currentChildGroup, currentChildHandler, currentChildOptions, currentChildAttrs));
                    }
                });
            }
        });
    }
```

