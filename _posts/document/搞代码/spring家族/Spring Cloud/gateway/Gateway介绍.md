### 配置文件注册路由
```
spring:
  cloud:
    gateway:
      routes:
        - id: jd
          uri: http://jd.com
          predicates:
            - Path=/jd
```

### 代码注册路由
```java
  @Bean
  public RouteLocator coustomRouteLocator(RouteLocatorBuilder routeLocatorBuilder) {
    return routeLocatorBuilder
        .routes()
        // basic proxy
        .route(r -> r.path("/bd").uri("http://baidu.com:80/").id("bd"))
        .build();
  }
```

### 监控端点
端点入口
```
GatewayControllerEndpoint
```

### 断言
SpringCloudGateway默认会提供很多断言
![](http://ww2.sinaimg.cn/large/006tNc79ly1g3w5jg9phnj30me082wg2.jpg)
- After:在某个时间之前
- Between:在某个时间中间
- Cookie:判断请求中是否包含指定的cookie
- Header:指定路由进行匹配
- Host:对指定的路由进行host匹配,成功则转发
- Method:队请求方法进行匹配,成功则转发
- Query:队请求的参数进行匹配,成功则转发
- RemoteAddr:对ip或者ip段进行匹配,成功则转发
- Path:对路径进行匹配,成功则转发


### 内置的Filter
Gateway中有许多内置的Filter,当然我们也可以通过自定义的方式来实现Filter,可以在转发的过程中修改请求.

- AddRequestParameter:在匹配的请求上添加header
- AddRequestParameter:在匹配的请求上添加上请求参数
- RewritePath:类似于zuul中的stripPrefix,功能更强大,可以提取uri中的内容进行替换
- AddResponseHeader:匹配的请求再返回时添加header
- StripPrefix:对url的前缀进行处理
- Retry:队请求进行重试,在Greenwich版本中uir无法携带参数
- Hystrix:对后端服务进行forback