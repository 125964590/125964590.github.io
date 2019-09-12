---
layout:     post
title:      Deployment控制器
subtitle:  云原生
date:       2019-09-12
author:     JBZM
header-img: img/deployment-controller.png
catalog: true
tags:
    - devops
    - k8s
---
# Deployment控制器

Kubernetes里面实现的第一个控制器就是Deployment,这个控制器应该也是我们使用最多的控制器了,对于paas而言水平扩展/收缩这个能力是必须具备的.Deployment就是尊徐一种**滚动更新**(rolling update)的方式来实现容器升级.

### ReplicaSet

这个能力的实现,依赖的主要是Kubernetes项目中的一个重要概念:**ReplicaSet**

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-set
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
```

从这个YAML中可以看到**一个ReplicaSet对象,其实就是副本数目的定义和一个Pod模板组成的.**也不难发现他的定义其实就是Deployment的一个子集.

### Deplioyment

**更重要的是Deployment控制器实际操纵的,就是这样的ReplicaSet对象,而不是Pod对象.**所以说一个pod的ownerReference就是ReplicaSet.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

具体的实现上,这个Deployment,与ReplicaSet以及Pod的关系如下图所示:

![关系](https://static001.geekbang.org/resource/image/ab/cd/ab4902a0437af4347bec520468c5e7cd.png)

Deployment通过操作ReplicaSet的个数来实现**水平扩展/收缩**和**滚动更新**两个编排动作.

可以尝试更改Deployment的属性来测试编排

```shell
$ kubectl rollout status deployment/nginx-deployment
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
deployment.apps/nginx-deployment successfully rolled out
```

> 这个命令可以直观的反映出我们在更改Deployment之后容器内部的变化

**在滚动更新之后可以使用命令查看新旧两个ReplicaSet的状态**

```shell
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
curl-6bf6db5c4f               0         0         0       46h
curl-78569f98bd               1         1         1       46h
nginx-deployment-5754944d6c   0         0         0       177m
nginx-deployment-7448597cd5   3         3         3       170m
nginx-set-59bd9cff8           3         3         3       17h
```

我们在使用k8s的滚动更新时一定要使用Pod的Health Check机制检查应用的运行装啊提,而且不是简单滴依赖容器的Running状态,要不然的话虽然容器已经变成了Running但是服务可能尚未启动,"滚动更新"的效果也就达不到了.

### 版本控制

为了进一步保证服务的连续性,Deployment Controller还会确保,在任何时间窗口内,只有指定比例的Pod处于离线状态.同事也会确保,在任何时间窗口内,只有指定比例的新的Pod被创建出来.

> 这两个比例的值是可以配置的,默认都是DESIRED值的25%

综上所述,展开一下Deployment,ReplicaSet和Pod的关系图

![展开关系图](https://static001.geekbang.org/resource/image/79/f6/79dcd2743645e39c96fafa6deae9d6f6.png)

如上图所示,Deployment的控制器,实际上控制的是ReplicaSet的数量一个每个ReplicaSet的属性.

一个版本对应的正式一个ReplicaSet,这个版本应用的Pod数量,则是由ReplicaSet通过他自己的控制器(ReplicaSet Controller)来保证的.

接下来我们尝试将nginx的版本号更改为一个仓库中不存在的版本,来观察版本升级的变化

更改后执行命令查看,发现新增了一个pod**nginx-deployment-7ff84c8bc9-55ndx**这个pod尝试创建但是由于指定的镜像不存在,这个pod是无法创建成功的

```shell
$ kubectl get pods
NAME                                READY   STATUS              RESTARTS   AGE
curl-78569f98bd-p98vc               1/1     Running             0          20h
nginx-deployment-7448597cd5-c4qlr   1/1     Running             0          3h19m
nginx-deployment-7448597cd5-n22lx   1/1     Running             0          3h17m
nginx-deployment-7448597cd5-zddpq   1/1     Running             0          3h18m
nginx-deployment-7ff84c8bc9-55ndx   0/1     ContainerCreating   0          3s
```

再来观察replica set的状态,可以发现新增了一个replica set **nginx-deployment-7ff84c8bc9**,他的期望个数是1,但是创建成功的个数为0该replica set更新失败,就不会进行接下来的滚动更新.

```shell
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
curl-6bf6db5c4f               0         0         0       47h
curl-78569f98bd               1         1         1       47h
nginx-deployment-5754944d6c   0         0         0       3h27m
nginx-deployment-7448597cd5   3         3         3       3h20m
nginx-deployment-7ff84c8bc9   1         1         0       12s
nginx-set-59bd9cff8           3         3         3       17h
```

### 版本回滚

如果想要将这次更新回滚,只需要执行下面的命令就可以

```shell
$ kubectl rollout undo deployment nginx-deployment
deployment.extensions/nginx-deployment rolled back
```

其实这次回滚操作,本质上就是把就得rs再次扩容到3个,新的rs收缩到0个

```shell
$ kubectl get replicasets.
NAME                          DESIRED   CURRENT   READY   AGE
curl-6bf6db5c4f               0         0         0       47h
curl-78569f98bd               1         1         1       47h
nginx-deployment-5754944d6c   0         0         0       3h38m
nginx-deployment-7448597cd5   3         3         3       3h31m
nginx-deployment-7ff84c8bc9   0         0         0       11m
nginx-set-59bd9cff8           3         3         3       17h
```

### 回滚指定版本

```shell
$ kubectl rollout history deployment/nginx-deployment --revision=2
```

在最开始启动Pod的时候添加了record,所以每次改变Deployment的时候都会有相应的记录,只需要选择需要回滚的版本号进行回滚即可

```shell
$ kubectl rollout undo deployment/nginx-deployment --to-revision=2
deployment.extensions/nginx-deployment
```

如果我们想让多次更新操作只执行一次滚动部署,那么就需要将Deployment的状态先暂停

```shell
$ kubectl rollout pause deployment/nginx-deployment
deployment.extensions/nginx-deployment paused
```

在完成更改之后重新开始则会激活在此期间对deployment的更改

```shell
$ kubectl rollout resume deploy/nginx-deployment
deployment.extensions/nginx-deployment resumed
```

## 总结

实际上Deployment是一个**两层控制器**.首先他通过**ReplicaSet**的个数来描述应用的版本;然后通过ReplicaSet的属性(比如replicas的值),来保证Pod的副本数量