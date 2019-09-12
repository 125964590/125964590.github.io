## 网卡配置
网卡配置非常重要,它关乎到我们的虚拟机与外界交互的出入口,在这里我只是放了一些关键的配置,具体教程小伙伴们可以点开文章最下方连接

### 宿主机
我们的宿主机实际上只有一个固定ip,通过物理网卡进出流量,通过网桥来搭建路由器,对虚拟网络提供入口
![](http://ws4.sinaimg.cn/large/006tNc79ly1g2juwb3n23j30c207mjrm.jpg)

#### 物理网卡

默认

```
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=eth0
UUID=8da786dd-1421-4dd1-b775-319fd52818a0
DEVICE=eth0
ONBOOT=yes
```



更改ONBOOT=yes

```
TYPE="Ethernet"
BOOTPROTO="static"
NAME="enp5s0f0"
UUID="4b557754-921d-46b0-b92c-b3b3cffcfec3"
DEVICE="enp5s0f0"
ONBOOT="yes"
BRIDGE="br0"
```
#### 桥接网卡
```text
BOOTPROTO=static
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=br0
UUID=242b3d4d-37a5-4f46-b072-55554c185ecf
DEVICE=br0
ONBOOT=yes
TYPE=bridge  # 将制定为桥接类型
IPADDR=192.168.1.24  # 设置IP地址
PREFIX=24               # 设置子网掩码
GATEWAY=192.168.1.1   # 设置网关
```

### 虚拟机
```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=eth0
UUID=56382371-6ea8-4adb-9ebf-5ef181ba6230
DEVICE=eth0
ONBOOT=YES
IPADDR=192.168.1.100
GATEWAY=192.168.1.1
PREFIX=24
DNS1=192.168.1.1
```

### 我遇到的一些坑

#### 有时需要去掉 type字段

网卡这边的坑说多不多说少还真不少,对于linux网络非常熟悉的小伙吧可以说是信手拈来,但是对于那些网络能力不是很强的人来说真的是折磨啊.

#### 不理解什么是网桥
最开始的时候我不清楚为什么要将物理网卡上绑定的ip换到新创建的网桥上,经过查阅资料还也是有了一定的了解.

让时空穿越到上世纪70年代，Ethernet被发明出来，需要用一个黑盒子将电脑连接起来；否则电脑和谁通信，和谁网线直连，这是点对点通信了，一点也不方便，和以太网的多路访问网络初衷相违背。以太网设计目标：电脑使用一个网络接口，可以同时与多台电脑通信，将电脑连接起来的黑盒子最先面世，称之为集线器，但我们更喜欢叫它Hub。这种集线器，通常有多个端口，可以接入多台电脑，这种黑盒子使电脑连接在一起成为一种可能，其内部工作原理，就是信号放大器。随着以太网的流行，越来越多的办公室使用它，网络变得越来越大，信号随着网线距离的变长，信号衰减的使得通信变得越发不可靠，同样需要可以将信号中继、放大的集线器。但这个集线器只需两个端口，一端连着一根网线，由于其本来的目的就是延伸网线，美其名曰：网桥，这个更像描述其功能性的一面，“牵线搭桥”之意。在集线器的基础上，添加了MAC地址学习功能，成为了交换机，这样可以避免集线器对所有帧都广播的弊病。但这些设备依然都是桥接设备，因为帧经过它们时，帧原封不动。但是随着无线局域网的诞生，关于桥接的定义被刷新，有线的Ethernet II 帧访问无线802.11时，帧格式发生了变化，但依然称AP（Access Point)为桥接设备，为何？因为一个广播帧可以无障碍通过AP，AP并没有分割广播域，所以依然是桥接设备。
>该回答引用知乎作者->'车小胖'

#### 网络概念模糊
这里不得不吐槽一下,自己了计算机网络都白学了,完全模糊的计算机网络概念,让在搭建网桥的时难受的一匹,这里写一下,通过配置网卡我学到了什么吧!
1. ip的分配方式;
2. DNS解析的原理;
3. 本地网关的作用;
4. Linux网络的配置方式;
5. 物理网卡在服务器上的作用;
6. 网桥的构建;
7. 虚拟网络如何使用.

### 参考文章
- [CentOS7安装虚拟机详解](https://github.com/jaywcjlove/handbook/blob/master/CentOS/CentOS7%E5%AE%89%E8%A3%85KVM%E8%99%9A%E6%8B%9F%E6%9C%BA%E8%AF%A6%E8%A7%A3.md#%E9%85%8D%E7%BD%AE%E7%89%A9%E7%90%86%E6%9C%BA%E7%BD%91%E7%BB%9C)
- [ubuntu18网卡配置教程](https://blog.csdn.net/peyte1/article/details/80509056)