# docker安装

## 安装docker
这里为了简化安装步骤直接使用DaoCloud封装好的安装命令
```
curl -sSL https://get.daocloud.io/docker | sh
```

## 设置镜像加速
如果不适用镜像加速很多镜像下载都会卡,这里统一使用DaoCloud的镜像加速
```
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
```
## 安装docker-compose
同样适用DaoCloud提供的方式进行安装
```
curl -L https://get.daocloud.io/docker/compose/releases/download/1.24.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```
配置执行命令
```
chmod +x /usr/local/bin/docker-compose
```

## 配置命令自动提示

安装bash-completion
```
yum install -y bash-completion
```