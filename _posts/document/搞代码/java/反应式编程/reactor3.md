# Reactor 3

## 介绍

1. Reactor是一个基于JVM,完全非阻塞的响应式编程框架,并且提供背压是的效率控制;
2. Reactor是完全基于Java8的函数式api;
3. Reactor提供可以组合使用的异步API,**Flux**(处理多个元素),**Mono**(处理单个元素);
4. Reactor提供基于**Netty**的非阻塞通信框架(HTTP,TCP,UDP).

### 阻塞会带来大量的性能损耗

如今的程序经常会面临高并发的考验,当并发量增大的时候我们能想到的最直观方法就是开启多个线程进行处理,对于JVM而言开启多线程有两种方式:

- Callable:创建一个可以会掉的函数交给新的线程进行执行;
- Futures:立刻返回一个Future<T>的返回值,但是实际需要等到任务执行结束才回拿到结果.

对于回调函数以及Futures,JVM现在已经做得很好了,但是对于代码的可读性以及编程的规范性破坏非常大,大篇幅的异步编程会造成(callback hell)

### 从命令式编程到响应式编程

类似recator这样的响应式库的目标就是要弥补经典的不足,此外还会关注一下几个方面:

- 可**编排性**(Composablility)以及**可读性**(Readablility);
- 使用丰富的**操作符**来处理**流**的数据;
- 在**订阅**(subscribe)之前什么都不会发生;
- **背压**(backpressure)消费者能够反向告知生产者生产内容的速度的能力;
- **高层次**的抽象,从而达到并发无关效果.

### 可编排性与可读性

随着异步任务数量的增多,复杂度的提高,编写和阅读的代价都会越来越大,**Reactor3**会提供丰富的编排API,从而直观的反应处理流程.

### 装配流水线

可以想象数据在响应式应用中的处理过程就像是流水线一样,材料从源头(**Publisher**)流出,最终被加工为成品,等待被推送到消费者(或者**Subscriber**),并且会有监控实时关注流水线的处理能力如果出现阻塞就减少投放.

### 操作符

在**Reactor**中,操作符就行是装配线中的工位.每一个操作符对**Publisher**进行处理,然后将其包装秤一个新的Publisher.就像一个链条,数据源自第一个**Publisher**顺着链条一直到最后一个

### subscribe()之前什么都不会发生

在**Reactor**中,当你创建了一条**Publisher**处理链,数据是不会生成的,实际上你创建的是一种抽象的处理流程.

真正**subscribe()**的时候需要将**Publisher**关联到一个**Subscriber**上,然后才回出发整个流程,这个时候**Subscriber**会向上发送一个**request**信号一直打到源头的**Publisher**.

### 背压

可以弹性的控制数据的生产以及消费

## Reactor核心特性

Reactor主要的依赖是**reactor-core**这是一个机遇Java 8 实现的响应式规范(Reactive Streams specification).

Reactor引入了实现**Publisher**的响应式类**Flux**和**Mono**,以及丰富的操作方式.

### 可编程式地创建一个序列

这一节,介绍如何通过相对应的时间(onNext,onError和onComplete)创建一个Flux或Mono.

#### [Generate](https://projectreactor.io/docs/core/release/reference/#producing.generate)

这是一种**同步地**产生各个值(state),最有用的方式就是记录一个状态值(state)在下发一个元素的时候根据这个值去生成需要下发的元素.

#### [Create](https://projectreactor.io/docs/core/release/reference/#producing.create)

最为一个更高级的创建Flux的方式,create既可以是同步的也可以是一步的,并且还可以每次发出多个元素.

#### [Handle](https://projectreactor.io/docs/core/release/reference/#_handle)

它与generate类似都是**同步**的,但是handler可以过滤一些元素

## [调度器(Schedulers)](https://projectreactor.io/docs/core/release/reference/#schedulers)

Reactor被认为是**并发无关**(concurrency agnostic)的,意思就是不强制要求任何并发的模型.我们可以根据自己的需要去实现线程池的调用模型.

