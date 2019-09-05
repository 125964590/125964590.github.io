## @import注解使用
先来看看官方给出的内容
```java
/**
 * Indicates one or more {@link Configuration @Configuration} classes to import.
 *
 * <p>Provides functionality equivalent to the {@code <import/>} element in Spring XML.
 * Allows for importing {@code @Configuration} classes, {@link ImportSelector} and
 * {@link ImportBeanDefinitionRegistrar} implementations, as well as regular component
 * classes (as of 4.2; analogous to {@link AnnotationConfigApplicationContext#register}).
 *
 * <p>{@code @Bean} definitions declared in imported {@code @Configuration} classes should be
 * accessed by using {@link org.springframework.beans.factory.annotation.Autowired @Autowired}
 * injection. Either the bean itself can be autowired, or the configuration class instance
 * declaring the bean can be autowired. The latter approach allows for explicit, IDE-friendly
 * navigation between {@code @Configuration} class methods.
 *
 * <p>May be declared at the class level or as a meta-annotation.
 *
 * <p>If XML or other non-{@code @Configuration} bean definition resources need to be
 * imported, use the {@link ImportResource @ImportResource} annotation instead.
 *
 * @author Chris Beams

 * @author Juergen Hoeller
 * @since 3.0
 * @see Configuration
 * @see ImportSelector
 * @see ImportResource
 */
```
### 在这里解释一下上面的一段话吧
通过声明的方式将一个或者多个类,通过@Configuration的方式导入.

提供一种方法,类似于Spring-XML中的'<import/>'配置方式.

允许导入使用的导入方式有@Configuration标注的类,标注了ImportSelector接声明了implementations

在使用的过程中,注入对象,是通过@Autowired的方式完成的

## 代码测试
```java
@Data
public class Dog {
  private String lol="lo";
}
```
随便创建一个类

```java
@SpringBootApplication
@Import(Dog.class)
public class GatewayServiceApplication {

  public static void main(String[] args) {
    SpringApplication.run(GatewayServiceApplication.class, args);
  }

}
```
通过@import将Dog.class包含进去

```java
@Slf4j
@Component
@AllArgsConstructor
public class TestHandler implements HandlerFunction<ServerResponse> {

  private final Dog dog;

  @Override
  public Mono<ServerResponse> handle(ServerRequest serverRequest) {
    log.info(dog.getLol());
    return ServerResponse.status(HttpStatus.OK)
        .contentType(MediaType.APPLICATION_JSON_UTF8)
        .body(BodyInserters.fromObject(dog.getLol()));
  }
}
```

通过@AutoWired的方式调用Dog,并且打印日志确认可以获得对象

### 测试结果
可以看到日志成功输出结果

![](http://ww3.sinaimg.cn/large/006tNc79ly1g3zik0t0zdj323e04igns.jpg)

## 源码分析
对于这类注解的解析位置我们可以通过查找对应注解名称+.class的方式进行定位

![](http://ww2.sinaimg.cn/large/006tNc79ly1g3zimmkc5rj30ve0u046j.jpg)

对源码进行追踪我们可以发现,在这个地方时对@Import中所声明的内容进行实例化的过程,通过Debug我们可以发现,此处代码会在Spring-Boot初始化的过程,进行Bean扫描,并且完成创建

## 总结
本文主要内容为一下几点:
- 解释@Import注解的作用;
- 演示@Import注解的使用方式;
- 定位@Import注解的扫描位置,以及实现原理和初始化过程.