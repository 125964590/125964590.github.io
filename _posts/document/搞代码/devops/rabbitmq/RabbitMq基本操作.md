## 写在前面
RabbitMq是一个消息中间件,目前我在生产中有如下几种使用方式:
- 异步处理
- 消息通讯

## RabbitMq的工作原理几种工作模式

### RabbitMq的工作原理
RabbitMq将自己定义为一个交换机,这也是在当初选择开发语言的时候选择一门交换机开发语言的原因,当然也正是因为选择了这门语言才让RabbitMq的部署和使用如此简单.

- publish:负责发布消息,在不同的工作模式下会选择将消息发送到交换机或者是队列中.
- exchange:负责处理消息,将消息发布到响应的队列中.
- queue:用来存放消息的队列.
- consumer:监听并消费队列中的消息.
- 
RabbitMq提供多种工作模式,在不同的生产环境我们需要选择适合自己的.

### RabbitMq的工作模式
- 基础模式
    - 声明队列之后将消息放到队列中并指定队列进行消费
- direct
    - 默认的模式消息会投递到相应的队列
- funout
    - 发布订阅模式,指定exchange并且发布消息,绑定了该exchange的queue都可以获得该消息
- topic
    - 匹配订阅,通过routingkey指定路由关键字,在消费端通过通配符来进行匹配消费

## 启动mq
为了方便学习这里直接使用docker启动一个单点的RabbitMq

### 启动RabbitMq
- mac操作系统
- docker--18v
- RabbitMq
![rabbitmq-version](https://ws1.sinaimg.cn/large/006rYg5Lly1fu2crhrhe0j30xg04l75m.jpg)

使用docker运行rabbitmq非常简单,只需要在运行的时候指定打开相应的端口号就可以了.
```
docker run -d --name myrabbitmq -p 5673:5672 -p 15673:15672 docker.io/rabbitmq:3-management
```

### 通过api连接mq

不多说直接上代码
```
 public static Connection GetRabbitConnection() {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUsername("guest");
        factory.setPassword("guest");
        factory.setVirtualHost("/");
        factory.setHost("localhost");
        factory.setPort(5672);
        Connection conn = null;
        try {
            conn = factory.newConnection();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return conn;
    }
```
这里是配置了一个连接工厂,通过这方法可以直接连接Rabbltmq

```
import pika

credentials = pika.PlainCredentials(username='guest', password='guest')
connection = pika.BlockingConnection(pika.ConnectionParameters(
    host='localhost', port=5673, credentials=credentials))
channel = connection.channel()
```
这里用python连接的需要使用pika,这也是官方推荐的连接方式.

[各种模式java测试代码](https://github.com/125964590/spring-boot-demo/tree/master/spring-boot-raggitmq-deom)


### 生产者以及消费者的事务处理
再生产这的地方目前对事务的控制不是特别关心
对消费者而言需要关心事务的问题,因为可能有多种原因需要考虑事务的问题.