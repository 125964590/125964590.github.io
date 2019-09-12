# SSH-DNS解析关闭
如果出现ssh登录缓慢的情况,需要关闭ssh-dns解析

```shell
（1）取消sshd服务的dns反向解析

       #vi /etc/ssh/sshd_config

（2）找到选项UseDNS ，取消注释，改为

       UseDNS no

（3）重启sshd服务

       /etc/init.d/sshd  restart
```