# 使用docker搭建es出现logstash无法写入FORBIDDEN/12/index read-only / allow delete (api)

## 问题描述
起因是logstash无法将数据写入到es中,通过查看日志发现logstash有如下输入
```
FORBIDDEN/12/index read-only / allow delete (api)
```
原因是es的磁盘空间超过了80%(默认值)

## 解决过程

### 发现问题
通过查看logstash发现无法向es中写入数据,索引变成了只读的状态,但是通过RestAPI进行测试发现,创建索引是可以的,往以前的索引中写数据是被禁止的并且有如下内容报错
```
FORBIDDEN/12/index read-only / allow delete (api)
```
查询资料发现,是因为es的磁盘空间超过了80%机会将索引编程只读的状态.

登录服务器进行查看磁盘空间有500G+喀什怀疑是Docker容器内部的资源有问题,但是想想也不对==docker在默认的情况下使用的资源和主机的资源是一致的==

使用esRestAPI进行查看集群状态发现总空间只有20G,已用空间达到17G

再次登录服务器查看

![](http://ws3.sinaimg.cn/large/006tNc79ly1g2dty07dwsj30c304v74u.jpg)

原来使用的是系统盘......也是醉了

### 解决问题
找到原因解决就比较容易了:
1. 更改docker默认文件仓库;
2. 重新创建镜像;
3. 测试查看镜像可用磁盘大小.

![](http://ws2.sinaimg.cn/large/006tNc79ly1g2du4q6arwj30lt01vt90.jpg)

完美解决~~~~

## 参考资料
- [es无法写入原因](https://discuss.elastic.co/t/forbidden-12-index-read-only-allow-delete-api/110282)
- [es官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/disk-allocator.html)
- [docker修改默认文件仓库](https://blog.51cto.com/forangela/1949947)