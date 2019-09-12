[TOC]

# vegas开发手册
欢迎加入vegas,本手册将带领你快速熟悉vegas系统.主要会分为一下几大模块:
- 入门篇
    - 账号申请;
    - 源码下载;
    - 开发环境准备;
    - 代码调试;
- 进阶篇之vegas-server
    - vegas框架整体介绍
        - 后端框架选型
        - 数据库设计
        - 模块职责介绍
    - vegas-platform
    - createtask-service
    - crontask-service
    - auth-service
    - admin-service
- 进阶篇之public-common
    - base-start
    - base-parent
    - base-common
- 进阶篇之发布部署
    - 推送镜像到docker hub

## 入门篇

### 账号申请
- 集团git:联系余超老师
- jumpserver:联系郑毅

### 源码下载
vegas所有的源码保存在集团git上,申请到账号后联系余超老师开通权限即可下载代码.

### 开发环境准备
- JDK 1.8+
- mvn 3.3.9+
- idea 2018+
- docker 18+

#### 配置host
配置host方法请自行百度

```
221.122.128.72  docker.registry
221.122.128.54  eureka-test.vegas.100tal.com
221.122.128.54  config-test.vegas.100tal.com
221.122.128.54  gateway-test.vegas.100tal.com
221.122.128.54  eureka.vegas.100tal.com
221.122.128.54  config.vegas.100tal.com
10.1.12.91  dev.vegas-server.com
```

#### 配置setting.xml文件
vegas项目组使用私有maven仓库,需要更换本地的setting.xml文件
> setting.xml文件请联系导师获取

### 代码调试
vegas所有的项目都是基于SpringBoot框架构建的,配置交给Apollo管理,所以启动项目前需要配置jvm参数,demo如下所示

```
-Denv=dev -Dapollo.configService=http://10.1.12.154:8080 -Ddev_meta=http://10.1.12.154:8080 -Dapp.id=VEGAS-PLATFORM-DEV
```
其中app.id需要根据启动的项目去替换

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4tns6dscqj305x08hdg1.jpg)

## 进阶篇之开发环境部署
开发环境部署可以由开发人员直接操作,内部逻辑实际上是将代码拉取到开发机进行打包发布.

### 脚本功能
帮助我们在开发环境(10.1.12.91)完成部署

### 注意事项
- 脚本中需要使用到ssh命令,所以请开发的同学提前配置好免密登录,[自动配置脚本](http://yi5.zheng@10.1.2.202/yi5.zheng/auto-sh.git)
- 使用如下脚本,可以将它添加到别名中,或者是直接调用
- 使用脚本需要配置host
```shell
#!/bin/bash

servers=$1
branch=$2

ssh root@dev.vegas-server.com "sh -x /home/jbzm/project/shell/gogogo ${servers} ${branch}"

exit 0
```
- 脚本需要的两个参数
	1. 需要部署的服务名称,可以使用','拼接
	2. 需要部署的分支

```
deploy-plus vegas-platform,crontask-service,createtask-service,admin-service jbzm-1.10
```

## 进阶篇之public-common
这个项目封装了vegas的一些基础组件和基础的父pom,下文内容主要介绍如何在项目中进行使用.

### base-parent
这个模块之所以用base-parent命名是因为,其内部做的事情就是对pom的定义,内部的嵌套关系如下所示
```
.
├── README.md
├── base-parent.iml
├── base-spring-boot
│   ├── base-spring-boot.iml
│   ├── base-spring-cloud
│   │   ├── base-spring-cloud.iml
│   │   └── pom.xml
│   └── pom.xml
└── pom.xml
```
#### 项目中使用
在具体的项目中,如果需要使用SpringBoot或者是SpringCloud相关的框架,则可以直接继承这里的pom,具体实例如下所示

```
    <parent>
        <groupId>com.talbrain.vegas</groupId>
        <artifactId>base-spring-cloud</artifactId>
        <version>2.0-SNAPSHOT</version>
    </parent>
```

### base-start
遇上一个模块类似,这个模块内部是一些自定义的start,稳定版的版本号,已全部在==base-spring-boot==中定义.

#### apollo-client-start组件
该组件为了适配vegas内部apollo的group不规范而存在的,由于同一套项目中有多个GroupName,所以在使用的时候必须动态的赋值.


##### 接入指南
目前由于Vegas-Server相关的项目有jar包干扰无法引入改Start,所有目前所服务的对象之后要paas->auth-service

###### 导入pom文件

导入pom之后就自动开启
```
        <dependency>
            <groupId>com.talbrain.vegas.start</groupId>
            <artifactId>apollo-client-start</artifactId>
            <version>2.0-SNAPSHOT</version>
        </artifactId>
```

###### 添加相关配置

在配置文件中添加如下配置,不同的namespace需要用','号分隔
```
apollo.branch = TEST1.rw.jdbc,TEST1.redis
```

#### web-tool-start
在项目pom中引入,如下图所示
```
        <dependency>
            <groupId>com.talbrain.vegas.start</groupId>
            <artifactId>web-tool-start</artifactId>
        </dependency>
```
##### 日志上报

###### 增加配置
log.enable默认为true,也就是说只要引用data.reqport.enable=true默认就会使用日志增强
```
web-tool:
  report:
    enable: true
    log:
      enable: true
```

###### 代码示例
```
    @GetMapping("lol/{p}")
    @LogTool
    public String lol(@PathParam("p") String p, @RequestParam String lol) {
        return "lol" + p + lol;
    }
```

###### 结果展示
```
2019-04-30 17:51:08.213 DEBUG 42251 --- [nio-8080-exec-4] c.t.v.start.report.log.util.LogAopUtils  : request url:http://localhost:8080/lol/test
2019-04-30 17:51:08.213 DEBUG 42251 --- [nio-8080-exec-4] c.t.v.start.report.log.util.LogAopUtils  : request methodcom.talbrain.vegas.start.AnnotationApplication
2019-04-30 17:51:08.213 DEBUG 42251 --- [nio-8080-exec-4] c.t.v.start.report.log.util.LogAopUtils  : request parameter:[null, haha]
2019-04-30 17:51:08.213 DEBUG 42251 --- [nio-8080-exec-4] c.t.vegas.start.report.log.aop.ImageAop  : this is web
2019-04-30 17:51:08.213 DEBUG 42251 --- [nio-8080-exec-4] c.t.vegas.start.report.log.aop.ImageAop  : run time :2
```

##### 动态更改日志级别
- 使用Spring提供的动态加载修改日志级别
- 集群模式下可以使用消息总线通信(该工能未提供)
```
   def changeLogLeave():
       body = {
           "configuredLevel": "DEBUG"
       }
       requests.request("POST", url="http://localhost:8080/loggers/com.talbrain.vegas", json=body)
```

###### 待完善功能 
- 开启条件控制
- 定义日志透传参数

##### 统一异常处理

###### 添加配置参数(该功能默认为开启状态)

```
web-tool:
  global-exception:
    enable: true
```

###### 代码实例

```
    @GetMapping("biz/{jbzm}")
    public String biz(@PathParam("jbzm") String jbzm) {
        throw new BizException(ErrorResult.create(68686868, "hah","jbzm-nb"));
    }
```

###### 结果展示

```
{
  "result": "hah",
  "message": "jbzm-nb",
  "code": 68686868
}
```

##### 带完善功能
- 目前根据业务情况只支持json的相应处理,后续将添加module的响应处理
- 开放自定义异常,以及自定义异常处理
- 对特定异常写入总线进行统计

#### ceph-S3-start组件
该组件封装了amazonaws的S3-API 增加了自动创建桶,自动获取桶名称等方法

##### 接入指南
**在需要接入的项目中添加pom文件**
```
      <dependency>
            <groupId>com.talbrain.vegas.start</groupId>
            <artifactId>ceph-S3-Start</artifactId>
        </dependency>
```

**添加配置文件**
```
oss-client:
  bucket-prefix: vegas3
  monitor:
    metadata-key: jbzm
    max-num: 2
    url: "http://39.97.118.225:8078"
    refresh-interval: 10000
  endpoint: "http://dawn.shareurl.facethink.com"
```

**代码示例**
通过==@Autowired==获取工具类,然后调用生成bucker name的方法
```
 @Autowired private OssUtils ossUtils;
 
  String url =
                uploadZyCloud.dotask(
                        ossClient,
                        ossUtils.getBucketName(),
                        fileName,
                        targetFilePath,
                        materialInfo.getTaskType());
```

> 该模块无需添加enable参数,默认为true,如果想禁用请添加eanble=false

##### 工作原理
该模块会在项目启动的时候获取保存在oss上的metadata判断当前bucket的偏移量,将当前偏移量缓存在本地.

项目运行中会根据==refresh-interval==参数去定时校验

## 进阶篇之发布部署

### 推送镜像到docker hub


需要先设置docker register配置
docker 容器化部署

- 添加配置文件
sudo vim /etc/docker/daemon.json

{
        "insecure-registries": ["docker.registry:30018"]
}
需要配置下这个   然后重启docker

-  然后再配置两条域名解析
221.122.128.72     docker.registry
221.122.128.72     auth-server
我们仓库的域名还没加到公网  所以前期先自己配置

服务器内网绑定如下host：

```
10.19.250.62   docker.registry
```

- 使用如下命令登录docker registry:

```
docker login docker.registry:30018
```

用户名： admin
密码：admin

或者

sudo docker login --username=admin docker.registry:30018
登录registry的用户名和密码是您注册容器服务时注册的用户名和密码。

你可以在镜像管理首页点击修改密码按钮修改docker login密码。
sudo docker login --username=admin docker.registry:30018