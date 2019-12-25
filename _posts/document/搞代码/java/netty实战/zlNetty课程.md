# zl Netty课程

## module 01 使用netty构建http服务器

初次了解netty,先使用netty构建一个我们熟悉的http服务器!!

### 项目介绍

使用netty创建一个接受请求的服务端,使用netty标准的构建模式:

- 创建服务端
  - 指定**boss线程组**和**worker线程组**
  - 指定通信模式(使用NIO)
  - 指定Initializer
  - 绑定端口
- 使用http编解码器
- 创建ServerHandler
  - 继承自**SimpleChannelInboundHandler**
  - 重写Read0方法,并将请求信息打印出来
  - 重写**注册/注销/活跃/不活跃**方法测试调用情况

```shell
.
└── netty
    └── zl01
        ├── TestServer.java
        └── handler
            ├── TestHttpServerHandler.java
            └── TestServerInitializer.java
```

### 执行结果

对服务端发送人已请求,返回Hello World

```shell
λ curl localhost:8899/lol
Hello World%
```

服务端可控制台输出

```
channelRegistered
channelActive
请求方法名:GET
请求路径:/lol
hello world
channelReadComplete
channelReadComplete
channelInactive
channelUnregistered
```

### 结果分析

通过控制台输出可以观察到netty接收到了请求并且打印出了相关信息

##module 02 使用netty搭建socket

## 

