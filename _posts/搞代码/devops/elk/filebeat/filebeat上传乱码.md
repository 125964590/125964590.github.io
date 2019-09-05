# 周报

## 周一

## 周二

## 周三

## 周四

## 周五

## 周末

## 本周工作完成

## 下周工作计划

## 客户心声

## 学习与反思## 写在前面
在ELK日志收集的使用中,filebeat上传的日志出现了乱码问题,这里记录一下解决问题的过程,由于我的bug原因比较特殊但是乱码现象网上还是很普遍的,多半是一些字符集编码的问题,我也尝试了进行字符集编码的更改,但是让然没有解决,最终通过修改日志打印格式解决了乱码的问题.

## bug描述
日志在kibana中展示出现了乱码(乱码截图已丢)

### 排查路线
首先检查对应服务器上的日志文件是否出现乱码
```shell
tail -100f /app/logs/base-search/base-search-info.log
```
查询之后没有发现乱码

开始怀疑是filebeat或者是logstash在解析日志的时候出现了乱码的问题,分别针对logstash和filebeat进行调试

#### logstash-开启控制态输出
由于是测试环境,再加上logstash进行重启或者停机并不会影响日志的保存(停机时间filebeat会检测到logstash已经down,之后停止发送日志).

对logstash的config文件进行重新编辑添加控制台输出
```config
output {
   stdout { 
    codec => json
   }
}
```
观察控制台打印直接输出乱码日志,怀疑logstash在解析日志时字符集出现不匹配,但是其他的输入源(filebeat)并没有出现乱码的情况,但是在config中也没有指定字符集编码.

选择查看上级数据源->filebeat

#### filebeat-debug模式
正常情况下filebeat不会吧传输的内容打印到日志中或者是控制带,但是filebeat给我们提供了debug模式,直接通过命令开启就行
```shell
filebeat -d -c config/filebeat-cocustom.yml
```
通过观察filebeat的日志输出存在乱码,这个时候人为基本找到bug了,因为通过之前查看系统日志发现控制台输出并没有乱码,但是filebeat中的内容出现了乱码,肯定是filebeat没有设置字符集编码

### 添加filebeat字符集编码
这个时候人为找到了bug的所在,开始进行百度,找到了解决方法开始修改编码.

但是经过各种尝试发现网上的修改字符集编码都不行,依然出现乱码的问题,这个时候怀疑是springboot在对日志文件输出的时候没有控制文件的输出格式,但是经过检查发现日志格式同样指定了utf-8的字符集编码.

#### 检查base-search-info.log文件编码情况
到这种情况就开始各种怀疑了,玄学找bug,通过file 命令查看文件编码格式发现,文件的编码格式也有问题
**下图是有问题的编码格式**
![文件编码格式](http://ws1.sinaimg.cn/large/006rYg5Lly1fya43hck3cj30ib026weu.jpg)

下图是正常的编码格式
![正常的编码格式](http://ws1.sinaimg.cn/large/006rYg5Lly1fya43hep5zj30c802k74k.jpg)

但是通过百度发现,修改文件编码格式依然解决不了上传时出现乱码的情况.

## 解决bug
再次查看日志发现,我的日志时带有颜色输出的,所以怀疑有可能是带有颜色输出导致filebeat在解析的时候出现了乱码
```
%d{yyyy-MM-dd HH:mm:ss} %clr{${LOG_LEVEL_PATTERN}} %clr{[%t]}{blue} %clr{%C.%M(%L)}{cyan} --> %m%n${sys:LOG_EXCEPTION_CONVERSION_WORD}
```
上面的命令为log4j2中设置日志输出颜色用的,最终将控制颜色的命令去除-->成功解决乱码问题.## 写在前面
在ELK日志收集的使用中,filebeat上传的日志出现了乱码问题,这里记录一下解决问题的过程,由于我的bug原因比较特殊但是乱码现象网上还是很普遍的,多半是一些字符集编码的问题,我也尝试了进行字符集编码的更改,但是让然没有解决,最终通过修改日志打印格式解决了乱码的问题.

## bug描述
日志在kibana中展示出现了乱码(乱码截图已丢)

### 排查路线
首先检查对应服务器上的日志文件是否出现乱码
```shell
tail -100f /app/logs/base-search/base-search-info.log
```
查询之后没有发现乱码

开始怀疑是filebeat或者是logstash在解析日志的时候出现了乱码的问题,分别针对logstash和filebeat进行调试

#### logstash-开启控制态输出
由于是测试环境,再加上logstash进行重启或者停机并不会影响日志的保存(停机时间filebeat会检测到logstash已经down,之后停止发送日志).

对logstash的config文件进行重新编辑添加控制台输出
```config
output {
   stdout { 
    codec => json
   }
}
```
观察控制台打印直接输出乱码日志,怀疑logstash在解析日志时字符集出现不匹配,但是其他的输入源(filebeat)并没有出现乱码的情况,但是在config中也没有指定字符集编码.

选择查看上级数据源->filebeat

#### filebeat-debug模式
正常情况下filebeat不会吧传输的内容打印到日志中或者是控制带,但是filebeat给我们提供了debug模式,直接通过命令开启就行
```shell
filebeat -d -c config/filebeat-cocustom.yml
```
通过观察filebeat的日志输出存在乱码,这个时候人为基本找到bug了,因为通过之前查看系统日志发现控制台输出并没有乱码,但是filebeat中的内容出现了乱码,肯定是filebeat没有设置字符集编码

### 添加filebeat字符集编码
这个时候人为找到了bug的所在,开始进行百度,找到了解决方法开始修改编码.

但是经过各种尝试发现网上的修改字符集编码都不行,依然出现乱码的问题,这个时候怀疑是springboot在对日志文件输出的时候没有控制文件的输出格式,但是经过检查发现日志格式同样指定了utf-8的字符集编码.

#### 检查base-search-info.log文件编码情况
到这种情况就开始各种怀疑了,玄学找bug,通过file 命令查看文件编码格式发现,文件的编码格式也有问题
**下图是有问题的编码格式**
![文件编码格式](http://ws1.sinaimg.cn/large/006rYg5Lly1fya43hck3cj30ib026weu.jpg)

下图是正常的编码格式
![正常的编码格式](http://ws1.sinaimg.cn/large/006rYg5Lly1fya43hep5zj30c802k74k.jpg)

但是通过百度发现,修改文件编码格式依然解决不了上传时出现乱码的情况.

## 解决bug
再次查看日志发现,我的日志时带有颜色输出的,所以怀疑有可能是带有颜色输出导致filebeat在解析的时候出现了乱码
```
%d{yyyy-MM-dd HH:mm:ss} %clr{${LOG_LEVEL_PATTERN}} %clr{[%t]}{blue} %clr{%C.%M(%L)}{cyan} --> %m%n${sys:LOG_EXCEPTION_CONVERSION_WORD}
```
上面的命令为log4j2中设置日志输出颜色用的,最终将控制颜色的命令去除-->成功解决乱码问题.