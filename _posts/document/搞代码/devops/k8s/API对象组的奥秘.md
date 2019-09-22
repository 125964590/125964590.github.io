# API对象组的奥秘

在kubernetes项目中,一个API对象在Etcd里的完整资源路径,是有Group(API组),Version(API版本)和Resource(API资源类型)三个部分组成的.

通过这样的结构,整个Kubernetes里的所有API对象,实际上就可以如下属性结构标识:

![API对象](https://static001.geekbang.org/resource/image/70/da/709700eea03075bed35c25b5b6cdefda.png)

这幅图中可以很清楚地看到**Kubernetes里API对象的组织方式,其实是层层递进的**

如果要声明创建一个CronJob对象,那么YAML文件应该这么写

```yaml
apiVersion: batch/v2alpha1
kind: CronJob
...
```

## 创建流程

在指定了API对象之后就开始想APIServer发起创建流程

![创建流程](https://static001.geekbang.org/resource/image/df/6f/df6f1dda45e9a353a051d06c48f0286f.png)

首先,当发起了创建CronJob的POST请求之后,我们编写的YAML信息就被提交给了APIServer