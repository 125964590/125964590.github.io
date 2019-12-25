# k8s安装

```shell
#!/bin/bash
systemctl stop firewalld.service
systemctl disable firewalld.service
setenforce 0
sed -i.bak 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
yum makecache
echo "net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1" > /etc/sysctl.d/k8s.conf
modprobe br_netfilter
sysctl -p /etc/sysctl.d/k8s.conf
swapoff -a
sed -i.bak "11s@UUID@#UUID@" /etc/fstab
echo "vm.swappiness=0" >> /etc/sysctl.d/k8s.conf
sysctl -p /etc/sysctl.d/k8s.conf
echo "KUBELET_EXTRA_ARGS=--fail-swap-on=false" > /etc/sysconfig/kubelet
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
chmod 755 /etc/sysconfig/modules/ipvs.modules
bash /etc/sysconfig/modules/ipvs.modules
lsmod|egrep "ip_vs|nf_conntrack_ipv4"
echo "{
\"registry-mirrors\": [\"http://f1361db2.m.daocloud.io\"],
\"exec-opts\": [\"native.cgroupdriver=systemd\"],
\"registry-mirrors\": [\"https://bsqmvrhu.mirror.aliyuncs.c1om\"]
}" > /etc/docker/daemon.json
systemctl restart docker
docker info|grep Cgroup
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
kubeadm init --config kubeadm.yaml --ignore-preflight-errors=all
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml
```

