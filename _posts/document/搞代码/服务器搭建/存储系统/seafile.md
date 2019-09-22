# Seafile

Seafile是一个类似百度云的云存储工具,支持mac,windows,linux,移动端以及web端.

## 服务端安装

直接使用docker搭建

```shell
docker run -d --name seafile \
  -e SEAFILE_SERVER_HOSTNAME=seafile.zhengyiwoaini.top \
  -e SEAFILE_ADMIN_EMAIL=125964590@qq.com \
  -e SEAFILE_ADMIN_PASSWORD=shuaizheng1995 \
  -v /data/seafile-data:/shared \
  -p 18000:80 \
  --restart always \
  seafileltd/seafile:latest
```

指定一个管理员账号即可,磁盘挂载在宿主机上,其内部维护数据关系使用的是mysql

## 客户端安装

客户端的安装直接上官网下载即可,提供多端选择

## 相关文档

- [官网文档](https://manual-cn.seafile.com/deploy/deploy_with_docker.html)

  