# kvm安装以及可视化

## kvm安装

这里直接把快速过一遍流程
**检查是否支持虚拟化**

```
cat /proc/cpuinfo | egrep 'vmx|svm'
```

**安装相关环境**
- qemu-kvm 主要的KVM程序包
- python-virtinst 创建虚拟机所需要的命令行工具和程序库
- virt-manager GUI虚拟机管理工具
- virt-top 虚拟机统计命令
- virt-viewer GUI连接程序，连接到已配置好的虚拟机
- libvirt C语言工具包，提供libvirt服务
- libvirt-client 为虚拟客户机提供的C语言工具包
- virt-install 基于libvirt服务的虚拟机创建命令
- bridge-utils 创建和管理桥接设备的工具

```shell
yum -y install qemu-kvm libvirt virt-install bridge-utils 
```
**检查服务是否正常运行**

```shell
systemctl start libvirtd
systemctl enable libvirtd
```
**安装虚拟机**
```shell
virt-install \
--virt-type=kvm \
--name=centos78 \
--vcpus=2 \
--memory=4096 \
--location=/tmp/CentOS-7-x86_64-Minimal-1511.iso \
--disk path=/home/vms/centos78.qcow2,size=40,format=qcow2 \
--network bridge=br0 \
--graphics none \
--extra-args='console=ttyS0' \
--force
```
基本上安装过程到这里就结束了,安装操作yum都已经封装好了全过程基本无坑.

## kvm可视化
kvm是我选择虚拟机平台,通过他来帮助我管理虚拟机,同样学习资料在文章最后面,这里主要讲一下我在操作是遇到的坑吧!!!

### 我遇到的坑

#### 第一大坑:概念模糊
如标题所说,初次使用kvm真的需要去补充一下相关的专有名词,将这些概念搞懂对于大家使用虚拟机甚至于理解linux操作系统都会带来帮助

#### 第二大坑:WebVirMgr搭建
平台的搭建比较容易按照官方说明一直走就可以,但执行完命令坑才刚刚开始

##### ssh无法登陆
当时没有截图所以这里图片有机会自补充,先通过文字描述记录一下,具体的Bug解决办法,在本文最后的学习资料中会给出的

**解决方法**
> 给安装WebVirMgr的服务器配置一个免密登录账号即可

##### 账号没有管理权限
> 将配置ssh的账号添加到root用户组,并切修改配置文件

##### noVNC无法使用
>本地安装onVNC之后打开6080端口(启动onVNC服务即可)
>如果是因为使用域名而导致的无法访问,那换成ip就可以了


### 学习资料
- [WebVirtMgr官方文档](https://github.com/retspen/webvirtmgr/wiki/Install-WebVirtMgr)
- [CentOS7安装教程(有点过时)](https://www.cnblogs.com/kevingrace/p/5739009.html)
- [handbook->kvm](https://github.com/jaywcjlove/handbook/blob/master/CentOS/CentOS7%E5%AE%89%E8%A3%85KVM%E8%99%9A%E6%8B%9F%E6%9C%BA%E8%AF%A6%E8%A7%A3.md#%E9%85%8D%E7%BD%AE%E5%AE%BF%E4%B8%BB%E6%9C%BA%E7%BD%91%E7%BB%9C)
- [bug解决](https://blog.xiaoben.li/p/497)
- [解决docer安装无法访问vnc](https://www.cnblogs.com/caidingyu/p/10868917.html)
- [docker安装](https://github.com/LY1806620741/web_kvm)