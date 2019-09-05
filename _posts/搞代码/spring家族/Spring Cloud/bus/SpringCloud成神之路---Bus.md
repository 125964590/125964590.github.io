## 概念
在微服务架构中,通常会使用轻量级的消息代理来构建一个公用的消息主题让系统中所有微服务实力都连接上来.由于主题中产生的消息会被所有实例舰艇和消费,所以我们称它为消息总线--翟永超(Spring-Cloud微服务实战)

### 作用
- 将消息路由到一个或多个地方
- 消息转化为其他的表现形式
- 执行消息的聚集,消息的分解,并将结果发送到他们的目的地,然后重新组合响应返回给消息用户
- 调用Web服务来检索数据
- 响应时间或错误
- 使用发布-订阅模式来提供内容或基于主题的消息路由

### 事件驱动模型
Spring的事件驱动模型中包含了三个基本概念:事件,事件监听者和事件发布者.

#### 事件
Spring重定义了事件的抽象类ApplicatioinEvent,他继承自JDK的EEventObject类.如果我们需要添加自定义事件,只需要继承ApplicationEvent.

#### 事件监听者
Spring中定义了事件监听者的接口ApplicationListener,他继承自JDK
的EventListener接口,同时指定了构造方法中的类型必须属于ApplicationEvent的某个子类.

#### 事件发布者
Spring重定义了ApplicationEventPublisher和ApplicationEventMulticaster两个接口来发布事件