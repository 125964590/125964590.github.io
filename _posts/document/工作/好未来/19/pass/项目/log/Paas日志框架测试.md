# Logback 对比 Log4j2

测试分为本地测试端和服务器端,分别对gateway网关进行日志输出的压测

## 同步日志,不打印日志

### logback(150并发,0间隔,循环50次)

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g799iq6avzj30vo028wep.jpg)

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g799j08p3hj313l0nomz5.jpg)

### log4j2(150并发,0间隔,循环50次)

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g79bj2sdqwj30vl02d3yq.jpg)

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g79bjnb03rj313u0p576a.jpg)

## 同步日志,打印日志(大小4k,循环输出100次)

### logback(150并发,0间隔,循环50次)

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g799nu4225j30vr020jrl.jpg)

![image-20190923105944045](https://tva1.sinaimg.cn/large/006y8mN6ly1g79a5lw6wej313t0ngjtf.jpg)

### log4j2(150并发,0间隔,循环50次)

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g79bdoxpg0j30vn028glt.jpg)

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g79bdwqr6wj313w0nn76a.jpg)

## 异步打印日志(大小4k,循环输出100次,仅输出文件)

### logback(150并发,0间隔,循环50次)

![image-20190923112435315](https://tva1.sinaimg.cn/large/006y8mN6ly1g79b9yvt39j30vm02uq37.jpg)

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g79acwngh2j313q0nkjte.jpg)

### log4j2(150并发,0间隔,循环50次)

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g79b8ioaauj30vs022wep.jpg)

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g79b8zrt2nj313o0nfmz3.jpg)

## 测试环境

### logback(50并发,0间隔,循环20次)

#### 不打印日志
![](https://tva1.sinaimg.cn/large/006y8mN6ly1g79opbngjwj30y702adg2.jpg)

#### 打印日志
![](https://tva1.sinaimg.cn/large/006y8mN6ly1g79oqf4b68j30y801qglt.jpg)

### log4j2(50并发,0间隔,循环20次)
#### 不打印日志

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g79oje87tjj30xz02kjrm.jpg)

#### 打印日志

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g79oiqfdl1j30yh02it8y.jpg)

### 打印日志每条30k输出1000次(一次请求30M)

#### log4j2(50并发,0间隔,循环20次)

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g7aealr4k7j30yb025dg2.jpg)

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g7aena98qlj31hp0u07ac.jpg)

#### logback(50并发,0间隔,循环20次)

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g7aek2a3fsj30ya0203yq.jpg)

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g7aek8mirtj31fz0u0dmf.jpg)

## 测试结果

### 对比响应时间

本地测试log4j2完爆logback(网关给的是2G堆内存)但是还是以测试环境为准:

1. 在不输出日志和仅输出400k日志的情况下log4j2的99%,95%相比logback的耗时都有所减少;
2. 同样开启异步日志的情况下输出30m日志的99%与95%差距在10倍以上.

### 对比jvm

对比30M日志输出时的JVM情况:

1. logback进行了大量的YCG去清理临时的临时变量,而log4j2的变化相对稳定;
2. logback的在异步处理的时候使用的是阻塞队列,日志输出量大导致写盘时间长队列堆积大使得处理日志时间过长;
3. 对比线程变化logback线程相对变化大于log4j2.

### 总结

1. 在线上环境关闭控制台输出会对性能有所提升;
2. 相同情况下对比,log4j2性能更优;
3. 在对日志输出场景比较多的情况下可以考虑将logback更换为log4j2;

## 附录

### 附录一 : logback vs log4j2

![](http://logging.apache.org/log4j/2.x/images/ResponseTimeAsyncLogging16Threads@8kEach.png)

### 附录二 : Sync vs Async

![](http://logging.apache.org/log4j/2.x/images/async-vs-sync-throughput.png)

### 附录三:log4j2特性

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g7aggcbaz4j31k80tmn3c.jpg)