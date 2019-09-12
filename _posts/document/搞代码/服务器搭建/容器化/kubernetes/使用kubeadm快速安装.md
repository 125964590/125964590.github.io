---
layout:     post
title:      使用kubeadm快速安装
subtitle:   服务器搭建
date:       2019-04-10
author:     JBZM
header-img: img/gateway多种获取方法测试.jpg
catalog: true
tags:
    - 服务器搭建
    - linux
	- k8s
---

# 使用kubeadm快速安装 

kubeadm是一种快速安装kubernetes的方法,最早的安装方法是完全通过二进制进行安装,但是随着k8s技术的发展出现了使用自身管理组件的方法去安装,说白了就是先安装一个kubeadm和一个kubelet.接下来的事情就交给kubeadm去做就好了.

## 安装kubeadm

使用Centos7安装需要手动添加源

```shell
#!/bin/bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet
systemctl start kubelet
yum install -y bash-completion*
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
systemctl enable kubelet.service
```

> 脚本参考博客[^1]
>
> [^1]: https://www.cnblogs.com/wzxmt/p/11160256.html

## 安装kubernets-master

现在我们的服务器已经拥有了kubeadm和kubelet,现在就可以让kubeadm来帮助我们完成kubernets的安装.

#### 编辑kubeadm.yaml

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 10.1.12.55
nodeRegistration:
  taints:
  - effect: PreferNoSchedule
    key: node-role.kubernetes.io/master
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.15.0
networking:
  podSubnet: 10.244.0.0/16
```

kubeadm本质上也是去创建pod,默认情况下kubeadm会帮我们生成如下yaml

```shell
$ cd /etc/kubernetes/manifests/
$ ls
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
```

这些yaml属于**static pod**(在kubelet启动的时候会自动开启)

在执行kubeadm的时候指定kubeadm.yaml就是为了指定生成yaml的方式,上文自定义了本地的ip并且使master节点可以部署pod

#### 开始安装

在开始安装之前需要准备一下docker的image,因为kubeadm默认回去国外仓库拉取镜像,由于天朝身处高位所以只能另寻出路.

```shell
#!/bin/bash
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.15.0
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.15.0
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.15.0
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.15.0
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.3.10
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.3.1
docker pull registry.cn-hangzhou.aliyuncs.com/mirror-suke/flannel:v0.11.0-amd64
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.15.0 k8s.gcr.io/kube-proxy:v1.15.0
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.15.0 k8s.gcr.io/kube-apiserver:v1.15.0
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.15.0 k8s.gcr.io/kube-controller-manager:v1.15.0
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.15.0 k8s.gcr.io/kube-scheduler:v1.15.0
docker tag registry.cn-hangzhou.aliyuncs.com/mirror-suke/flannel:v0.11.0-amd64 quay.io/coreos/flannel:v0.11.0-amd64
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.3.1 k8s.gcr.io/coredns:1.3.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.3.10 k8s.gcr.io/etcd:3.3.10
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1 k8s.gcr.io/pause:3.1
```

做法很简单就是上阿里云吧需要的镜像下载下来然后打上我们需要的tag

接下来直接执行命令安装即可

```shell
$ kubeadm init --config kubeadm.yaml
```

在执行结果里会让我们保存一个token和修改命令的执行权限:

- 返回给我们的命令可以用来在其他服务器增加node
- 修改权限是为了让我们可以使用kubectl

```shell
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### 安装网络插件

安装完成后执行命令查看状态

```shell
$ kubectl get nodes
NAME         STATUS   ROLES    AGE     VERSION
k8s-master   Ready    master   3h14m   v1.15.3
k8s-work01   Ready    <none>   101m    v1.15.3
k8s-work02   Ready    <none>   76m     v1.15.3
```

如果出现**NoReady**是因为我们还需要安装网络插件

```shell
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml
```

安装完成之后就会变成Ready的状态

现在我们的master基本上就已经安装完成了.

## 安装kubenetes-work

work节点安装相对容易很多,原理和master一样:

1. 安装kubeadm和kubelet
2. 下载需要的镜像
3. 将master返回的命令执行

执行命令查看即可

```shell
$ kubectl get nodes
NAME         STATUS   ROLES    AGE     VERSION
k8s-master   Ready    master   3h14m   v1.15.3
k8s-work01   Ready    <none>   101m    v1.15.3
k8s-work02   Ready    <none>   76m     v1.15.3
```

