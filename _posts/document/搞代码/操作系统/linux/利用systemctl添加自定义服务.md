# 利用systemctl添加自定义服务

对于Centos 7而言我们可以将我们需要运行的软件当做一个后台服务运行

```shell
[Unit]
Description=frp client

[Service]
ExecStart=/opt/frp_0.29.0_linux_amd64/frpc -c /opt/frp_0.29.0_linux_amd64/frpc.ini

[Install]
WantedBy=multi-user.target
```

上面这个配置文件是最简单的一个配置,我们需要把这个文件放在**/etc/systemd/system/**下面,并且文件需要用**.service**结尾这样我们的应用就可以用下面这种方式启动

```shell
$ systemctl restart frpc.service
```

