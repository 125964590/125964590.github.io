## 简单介绍下Ribbon
Ribbon是Netflix发布的负载均衡器,默认提供很多负载均衡的算法,在Spring Cloud的体系中主要的作用是对服务的消费(从注册中心中获取服务器的信息,之后经过负载均衡的算法去调用相应的服务器)
## Ribbon 的工作方式
![这里写图片描述](https://img-blog.csdn.net/201804231811236?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NjYzODk5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
该方法通过RESTful进行请求,简单点讲就是使用了一个拦截器,在调用RestTemplate的时候拦截url进行解析完成服务的调用.
![这里写图片描述](https://img-blog.csdn.net/20180423214519458?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NjYzODk5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
## 为服务消费者整合Ribbon
为服务添加如下配置
```java
@Configuration
public class MyConfig {
    @Bean
    @LoadBalanced
    public RestTemplate loadTemplate() {
        return new RestTemplate();
    }

    @Primary
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```
@LoadBalanced该注解表示在加载RestTemplate的时候引入LoadBalance的加载

- Ribbon就是通过RESTful对服务进行调用,经过LoadBalanced加工之后

```java
    @Autowired
    @LoadBalanced
    private RestTemplate restTemplate;
    @Autowired
    DiscoveryClient discoveryClient;



    @GetMapping("consumer")
    public String dc() {
        return restTemplate.getForObject("http://eureka-client/dc", String.class);
    }
```
- Ribbon在实现REST请求的时候加了一层连接器,将内部的服务名解析了出来.
## 开始测试
- 启动两个Eureka-Client(负载均衡)两个Eureka-Server(高可用注册中心)启动一个Ribbon消费者
![这里写图片描述](https://img-blog.csdn.net/20180423215147634?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NjYzODk5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
- 请求接口检测负载均衡的效果
![这里写图片描述](https://img-blog.csdn.net/20180423215244689?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NjYzODk5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](https://img-blog.csdn.net/20180423215252747?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NjYzODk5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
如果两次请求出现1 和 2 来回切换则表示负载均衡配置成功.

# 这里就先这样吧我也是醉了,这个基本书上怎么讲的都是Ribbon的源码啊............需要的时候再看吧.