# 理解独立的Spring应用

SpringBoot它内置了web容器并且可以直接生成可执行的jar包,这里主要讨论可执行jar包中的内容

## 创建SpringBoot应用可知行JAR

SpringBoot之所以可以使用Maven打包生成可执行JAR包是因为他其中包含了maven的工具插件

```xml
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
```

他会帮我们完成打包,这里对其执行的过程以及原理不做过多的赘述,接下来对打包后生成的目录结构进行分析

```text
.
├── BOOT-INF
│   ├── classes
│   │   ├── application.properties
│   │   ├── thinking
│   │   │   └── in
│   │   │       └── spring
│   │   │           └── boot
│   │   │               ├── autoconfigure
│   │   │               │   └── WebAutoConfiguration.class
│   │   │               ├── config
│   │   │               │   └── WebConfiguration.class
│   │   │               └── firstappbygui
│   │   │                   └── FirstAppByGuiApplication.class
│   │   └── uml
│   │       └── Launcher.ucls
│   └── lib [60 entries exceeds filelimit, not opening dir]
├── META-INF
│   ├── MANIFEST.MF
│   ├── maven
│   │   └── thinking-in-spring-boot
│   │       └── first-app-by-gui
│   │           ├── pom.properties
│   │           └── pom.xml
│   └── spring.factories
└── org
    └── springframework
        └── boot
            └── loader [15 entries exceeds filelimit, not opening dir]
```

- BOOT-INF/classes存放编译后的class文件;

- BOOT-INF/lib目录存放应用依赖的JAR包;

- META-INF/目录存放应用相关的元信息,如MANIFEST.MF文件;

- org/目录存放SpringBoot相关的class文件.

对比maven打包生成的原jar解压后的文件可以发现repacking之后内部增加了很多东西,所以SpringBoot的启动和这些文件有关.

### FAT JAR和WAR执行模块--spring-boot-loader

我们在使用*java -jar*命令执行SpringBoot编译之后的jar包时实际上是遵循java的原始逻辑也就是说SpringBoot是按照标准引导的JAR执行**他的打包规范也是按照标准JAR来的**

打开**/META-INF/MANIFEST.MF**文件

```config
spring-boot-maven-pluginManifest-Version: 1.0
Implementation-Title: 《Spring Boot 编程思想》图形化构建 Sp
 ring Boot 应用
Implementation-Version: 0.0.1-SNAPSHOT
Built-By: zhengyi
Implementation-Vendor-Id: thinking-in-spring-boot
Spring-Boot-Version: 2.0.2.RELEASE
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: thinking.in.spring.boot.firstappbygui.FirstAppByGuiApplic
 ation
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Created-By: Apache Maven 3.6.0
Build-Jdk: 1.8.0_171
Implementation-URL: https://projects.spring.io/spring-boot/#/spring-bo
 ot-starter-parent/first-app-by-gui
```

> 根据上面这个文件我们可以知道代码的启动以来**JarLauncher**,接下来我们尝试用java去引导**JarLauncher**

```verilog
→ java org.springframework.boot.loader.JarLauncher
.   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.0.2.RELEASE)

2019-08-19 17:02:17.713  INFO 10987 --- [           main] t.i.s.b.f.FirstAppByGuiApplication       : Starting FirstAppByGuiApplication on bogon with PID 10987 (/Users/zhengyi/Documents/zy/MyJava/PublicProject/thinking-in-spring-boot-samples/spring-boot-2.0-samples/first-app-by-gui/target/tmp/BOOT-INF/classes started by zhengyi in /Users/zhengyi/Documents/zy/MyJava/PublicProject/thinking-in-spring-boot-samples/spring-boot-2.0-samples/first-app-by-gui/target/tmp)
```

这里不难发现项目成功的运行了起来,也就是说我们在运行springboot所生成的JAR包时实际上是启动了**JarLauncher**这个类,之后再由**JarLauncher**去引导**MANIFEST.MF**中*Start-Class:*所表示的类

> 接下来尝试用java去引导**FirstAppByGuiApplication**看看会有什么发生

```verilog
→ java thinking.in.spring.boot.firstappbygui.FirstAppByGuiApplication                                                                                                   [d2ef2d7]
Exception in thread "main" java.lang.NoClassDefFoundError: org/springframework/boot/SpringApplication
	at thinking.in.spring.boot.firstappbygui.FirstAppByGuiApplication.main(FirstAppByGuiApplication.java:25)
Caused by: java.lang.ClassNotFoundException: org.springframework.boot.SpringApplication
```

不难发现JVM启动失败了,这里找不到SpringApplication这个类

> 这里报错是因为Spring相关的JAR文件都存放在了**BOOT-INF/lib**需要执行Class Path才可以进行访问.

### JarLauncher的实现原理

