# Pod资源调度

## 写在前面

在系统的迭代过程整用户对于CPU/GPU的需求变多,物理机也随着增多,如何将pod在各个物理机之间合理的调度使我们现在需要考虑的问题,对于GPU需求大的会走到GUP上某些内存优先的需要部署在内存型服务器等等.

好在k8s对于pod的调度提供了非常好的支持,本篇文章主要讲述k8s自身的几种pod调度方式.

## Deploymen/RC:全自动调度

顾名思义使用Deployment对象部署的pod所有的资源调度交给k8s自身进行调度

## NodeSelector:定向调度

基于Node标签方式的调度,可以吧集群中具有不同特点的Node贴上不同的标签,再部署应用是就可以根据应用的需求设置NodeSelector来制定Node的调度范围.

在使用NodeSelector的时候k8s本身也会给Node指定一些默认的标签,所以我们在使用的时候可以直接指定默认的标签.

但是**NodeSelector**是一种固定的调度策略,随着集群的复杂度增加调度算法的智能化增加亲和性调度(NodeAffinity)将代替NodeSelector.

### 给节点添加标签

```shell
kubectl label node k8s-work01 type=cpu
```
### yaml文件
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-node-selector
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
      nodeSelector:
        type: cpu
```

## NodeAffinity: Node亲和性调度

NodeAffinity意为Node亲和性调度策略,是用于替换NodeSelector的全新调度策略.目前有两种节点亲和性表达.

- RequiredDuringSchedulingIgnoredDuringExecution:必须满足指定规则;
- PreferredDuringSchedulingIgnoredDuringExecution:强调有限满足指定规则;

**NodeAffinity**规则设置的注意事项如下:

- 如果同时定义了nodeSelector和nodeAffinity,那么必须两个条件都得到满足,Pod能最终运行在指定的Node上;
- 如果nodeAffinity指定了多个nodeSelectorTerms,那么只需要其中一个能够匹配成功即可;
- 如果nodeSelectorTerms中有多个matchExpressions,则一个节点必须满足所有的matchExpressions才能运行该pod.

## PodAffinity:Pode亲和性调度

和上面node亲和性调度类似这里也是根据亲和与互斥进行调度,根据label去选择pod,但是在选择的时候也会依赖Node的一些信息.

如果在具有标签X的Node上运行了一个或者多个符合条件Y的Pod,那么Pod应该(如果是互斥的情况,那么就变成了拒绝)运行在这个Node上.

这里X指的是集群中的节点、机架。区域等概念，通过Kubernetes内置节点标签中的key来进行声明.这个key的名字为topologyKey,以为表达节点所述的topology范围.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-base
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-os
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-os
  template:
    metadata:
      name: nginx-os
      labels:
        app: nginx-os
    spec:
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - nginx
              topologyKey: kubernetes.io/os
      containers:
        - name: nginx
          image: nginx
```

## Taints和Tolerations(污点和污点容忍)

前面介绍的NodeAffinity节点亲和性,是在pod上定义的一种属性,是的Pod能够被调度到某些Node上运行(优先选择或者强制要求)Taints正好相反---他是让Node拒绝运行Pode.

Taints和Tolerations需要配合使用,让Pod来避开那些不适合的Node.在一个Node上设置了一个或者多个Taints之后除非Pod能够容忍这些污点否则Pod将无法在上面运行.Tolerations是Pod的属性能够让Pod运行在有污点的机器上(注意是允许而不是必须).

**设置Taints信息**

```shell
$ kubectl taint node k8s-work01 kind=work01:NoSchedule
```

