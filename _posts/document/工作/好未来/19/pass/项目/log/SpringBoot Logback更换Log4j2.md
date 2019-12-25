# SpringBoot Logback更换Log4j2

在默认情况下SpringBoot使用的是Logback进行日志处理,但是根据上一篇[文章](http://seafile.zhengyiwoaini.top/lib/4e31cd37-be13-4ae0-abd8-63591e0f4562/file/工作/好未来/19/pass/log/Paas日志框架测试.md)的对比可以看出来Log4j2的性能是明显高于Logback,这里不过多的讨论为什么更换为Log4j2,只对具体的操作进行描述.

## 添加Log4j2依赖并派出Logback

再倒入Log4j2之前必须将Logback的依赖排除掉否则,在启动的时候Sl4j无法寻找对应的日志框架,由于**SpringBoot默认使用的是Logback**所以需要查询依赖树将Spring内部的引用exclude.

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g7al8qdiskj30gt02nwew.jpg)

实际上需要排出的情况会比较多,根据引用的不同包情况也是不一样的,这里就不一一例举,具体的操作都是一样的

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

添加Log4j2与异步支持

> 这里说明一下,根据官方推荐和实际测试结果开启异步日志的性能会有所提升,但是测试的范围比较单一,具体变化还是要等待实际项目中的体现**尤其是使用了父级pom时需要将父级pom中的依赖排除**

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>
        <dependency>
            <groupId>com.lmax</groupId>
            <artifactId>disruptor</artifactId>
            <version>3.3.7</version>
        </dependency>
```

### 添加Log4j2相关配置

在resources下添加(位置可以相对调整)log4j2.xml文件并且在Spring的application中进行告知,**在替换的时候需要更改SERVER_NAME**

```properties
logging.config=classpath:log4j2.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration status="INFO" monitorInterval="60">
    <Properties>
        <property name="LOG_HOME" value="./logs"/>
        <property name="SERVER_NAME" value="gateway"/>
        <Property name="LOG_PATTERN">%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n
        </Property>
    </Properties>
    <appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout charset="UTF-8" pattern="${sys:LOG_PATTERN}"/>
        </Console>
        <RollingFile name="INF" fileName="${LOG_HOME}/${SERVER_NAME}.%d{yyyy-MM-dd}.%i.log"
                     filePattern="${LOG_HOME}/pack/${SERVER_NAME}_log4j2_inf-%d{yyyy-MM-dd}.log.zip"
                     immediateFlush="true">
            <PatternLayout charset="UTF-8"
                           pattern="${sys:LOG_PATTERN}"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
            </Policies>
            <DefaultRolloverStrategy>
                <Delete basePath="./logs/pack/">
                    <IfLastModified age="30d"/>
                </Delete>
            </DefaultRolloverStrategy>
        </RollingFile>
    </appenders>
    <Loggers>
        <Root level="INFO">
            <AppenderRef ref="INF"/>
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</configuration>
```

## 开启异步日志

异步日志在输出大量日志时会减少阻塞的情况Log4j2官方的推荐是直接开启全量异步日志,我在这里也是这么做的,通过添加JVM参数即可

```java
-Dlog4j2.contextSelector=org.apache.logging.log4j.core.async.AsyncLoggerContextSelector
```

