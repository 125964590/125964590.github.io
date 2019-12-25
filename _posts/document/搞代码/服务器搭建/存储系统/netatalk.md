---
layout:     post
title:      Centos7 配置netatalk搭建mac Time Machine
subtitle:   服务器搭建
date:       2019-09-11
author:     JBZM
header-img: img/gateway多种获取方法测试.jpg
catalog: true
tags:
    - 服务器搭建
    - linux
    - mac
---

# Centos7 配置netatalk搭建mac Time Machine

mac的Time Machine是一个备份的功能,他会增量的为我们备份系统,如果你的mac丢失了或者是进水了(彻底死亡)这是你有Time Machine的时间备份,那么就可以去苹果商店买一个新的mac使用Time Machine将新的机器还原到你当初的状态.

使用Time Machine备份有两种方式:

- 磁盘备份
- mac的网络文件系统afp

磁盘备份,这个很简单只需要我们外接一个存储硬盘并且格式化成Time Machine需要的格式即可.

afp网络磁盘这个需要mac自己的网络存储硬件支持,但是如果我们单纯是为了做Time Machine的备份去买一个网络存储硬件确实有点奢侈,接下来本文主要介绍如何在Centos7上通过netatalk模拟afp.

## netatalk搭建afp网络

Centos7上没有直接提供的rpm可以使用这里需要手动安装,手动安装有两种方式**构建rpm包**和**源码安装**

### 完成rpm包的编译

```shell
$ yum install mock -y
$ useradd -m mock -g mock
$ su - mock
$ wget http://www003.upp.so-net.ne.jp/hat/files/netatalk-3.1.11-1.4.fc29.src.rpm 
$ mock -r /etc/mock/epel-7-x86_64.cfg --rebuild netatalk-3.1.11-1.4.fc29.src.rpm
```

### 安装并配置netatalk

```shell
$ cd /var/lib/mock/epel-7-x86_64/result
$ yum localinstall netatalk-3.1.11-1.4.el7.x86_64.rpm -y
$ vim /etc/netatalk/afp.conf
```

### 修改配置文件

```
[Global]
 log level = defalut:war
 log file = /var/log/afpd.log    # 存放日志
 spotlight = yes

[My Time Machine Volume]
 path = /home/macbackup/to/backup    # afp需要共享的文件路径(需要手动创建)
 time machine = yes
 spotlight = no
 ea = auto
 valid users = macbackup    # 指定登录的用户(需要手动创建,并且拥有上面path的读写权限)
```

### 创建用户,netatalk登录时需要使用

```shell
$ useradd macbackup
$ passwd macbackup
$ mkdir -p /home/macbackup/to/backup
$ chown -R macbackup /home-back/
```

### mac连接afp

#### 打开**finder**选择**connect server**

输入你配置的账号密码
![](http://ww3.sinaimg.cn/large/006y8mN6ly1g6741t71cjj30d7067glq.jpg)

#### 登录Time Machine就会发现新增的磁盘选择即可

![](http://ww3.sinaimg.cn/large/006y8mN6ly1g67445hks7j30mc07qdhp.jpg)

## 相关文档

- [netatalk官方文档](http://netatalk.sourceforge.net/3.1/htmldocs/afp.conf.5.html)
- [教程01](https://blog.51cto.com/penguintux/2318250)
- [教程02](https://blog.csdn.net/baidu_38741636/article/details/79241638)