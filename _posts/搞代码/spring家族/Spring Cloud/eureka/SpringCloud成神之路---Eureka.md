# SpringCloud成神之路(一)------eureka
## 服务的注册与发现
服务提供者,服务消费者,服务发现组件这三者之间的关系大致如下:
服务提供者消费者:

- 个个微服务在启东时,将自己的网络地址等信息注册到服务发现组件中,分局无发现组件会存储这些信息.
- 服务消费者可从服务发现组件查询服务提供者的网络地址,并使用该地址调用服务提供者的接口.
- 个个服务之间的通信方式保持一致,如续约制度,心跳制度等.

服务发现组件:

- 服务注册表:是服务发现组件的核心内容,用来存放个个服务的基本信息(服务名称,地址IP端口号等信息)
- 注册与发现:
  - 注册的意思是当服务启动的时候会连接注册中心,注册中心将服务的基本信息进行保存的过程.
  - 发现的意思是通过名称和端口号的查询可以找到该服务所对应的机器
- 服务检查:指的是一个服务已经注册到了注册中心,注册中心会定期对服务进行检查,以保证该服务是一致处于可以被调用的状态.
### 讲一下bootstrap(我已开始看到挺蒙的)
- bootstrap.yml
  - 这是一个可以被继承的配置文件主要给application.yml提供属性(其中存放的参数主要是SpringCloud)
- RetryLister工厂
  - 如果需要使用多个[RetryLister](http://cloud.spring.io/spring-cloud-static/Edgware.SR3/single/spring-cloud.html#_retrying_failed_requests)可以使用这个工厂去定义
![分层介绍](https://img-blog.csdn.net/20180423022013135?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NjYzODk5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 简单的说一下这个东下吧
官方给的文档说在使用Spring Cloud的时候推荐使用这种方式对配置文件进行整合,该配置文件与application.yml配置文件是存在集成关系的,子类可以集成服务类的配置文件,在子类中也可以重新定衣父类的配置文件(具体生产中没有使用过,目前还没有涉及到).

## Spring Cloud-----eureka
学习Spring Cloud的组件就先从注册中心开始吧(我的目的也是如此,当务之急是先搭建出来一个注册中心和高可用的服务)

### 简单介绍
Eureka的学习过程简单的总结一下,就是SpringCloud通过统一的封装将Eureka,Zookeeper,Canso等注册中心封装成了一套统一的API在使用的时候可以仅仅通过更换maven包实现代码零更改的切换.

- Eureka-Server
  - 主要是用来提供注册的注册中心,默认自己也会注册自己(实现高可用).
- Eureka-Client
  - 客户端存在于每个服务,可以向注册中心注册自己.

### Eureka---Server搭建
单独创建一个项目引入pom文件,先搭建一个单点的注册中心
```
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka-server</artifactId>
		</dependency>
	</dependencies>  

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR1</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
```
在启动类添加相应注解
```

@SpringBootApplication
@EnableDiscoveryClient
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```
添加相应的配置
```
spring:
  application:
    name: eureka-server   #服务名称
server:
  port: 10010   #制定端口号
# eureka配置
eureka:
  instance:
    hostname: localhost   #标识注册的ip
  client:
    serviceUrl:
      defaultZone: http://localhost:10009/eureka    #向注册中心注册的节点
  client:
      register-with-eureka: false   #禁止向eureka注册自己
      fetch-registry: false   #是否从eureka获取注册信息
  server:
    enable-self-preservation: false   #禁止使用安全检查
```
这里启动服务端,	访问端口10010注册中心没有内容展示说明注册中心启动成功
![这里写图片描述](https://img-blog.csdn.net/20180423132456664?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NjYzODk5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### Eureka---Client搭建
Client的搭建比较简单,引入pom文件,在启动类上添加开启Client的注解,修改yml的配置即可完成服务的注册
```

@SpringBootApplication
@EnableDiscoveryClient
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```
修改相应配置文件
```
server:
  port: 10011
spring:
  profiles:
    active: second
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:10010/eureka,http://localhost:10009/eureka
```
启动Client服务观察注册中心的状态
![这里写图片描的述](https://img-blog.csdn.net/20180423134729126?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NjYzODk5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
在服务注册中出现了客户端的注册.说明注册成功

### Eureka高可用搭建
Eureka的高可用搭建非常简单,在之前的配置文件中定义了两个false这两个,表示的就是让Eureka不会自己注册自己,Eureka的高可用就是可以向注册中心注册自己,是的两台以上的服务形成高可用.
**其实上图就一个高可用的注册,我在截图的时候偷懒了,从上图可以直接看到Server属性后面可以看到有两台服务器,一台代表着自己另一台是其他的注册中心.

## Eureka 服务调用的方式
### Eureka 元数据
Eureka元数据有两种一种是自定义元数据,一种是标准元数据
- 标准元数据包括主机名,IP地址,端口号,等健康信息.
- 自定义元数据我们可以通过eureka.instance.metadata-map来自定义元数据,存储方式是以keyvalue的形式.
Eureka元数据断点信息
![这里写图片描述](https://img-blog.csdn.net/20180423150433830?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NjYzODk5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
这是请求之后返回的断点元数据,通过元数据可以发现我们仅仅通过服务的名称就可已得到服务的端口号和IPD地址
	通过json观察我们可以发现返回的是一个数组.	
```
     @GetMapping("discovery")
    public Object look() {
        List<ServiceInstance> instances = discoveryClient.getInstances("eureka-client");
        instances.forEach(x->{
            System.out.println(x.getHost());
            System.out.println(x.getUri());
            System.out.println(x.getPort());
        });
        return discoveryClient.getInstances("eureka-client");
    }
```
![这里写图片描述](https://img-blog.csdn.net/2018042315160396?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NjYzODk5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](https://img-blog.csdn.net/20180423151721287?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NjYzODk5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
解析出来的结果,这里我启动了两个同名的服务进行测试,可以看到获取到了两个服务,这个时候我们就完成了Eureka的基本功能----服务的注册与发现,接下来就是对服务进行调用,对负载均衡的使用(在之后的章节进行讲解)

## 最后在说些什么吧
Spring Cloud对服务的调用封装了多种方式,这里使用的Discovery是属于很底层的方法,其他的方法还有LoadBalanceClient,Ribbon,Feign等些在之后会去学的.总之机制都是一样的,还有很多写的不够细节,没办法啦~毕竟写这个东西只是为了加强学习的记忆并不是为了给谁看(╯▽╰).