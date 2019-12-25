# Ingress安装

## 写在前面

这篇文章本来想记录一下配置Ingress的过程,但是基本上是按照官方文档操作的,所以在文末就直接附上文档的地址.这篇文章主要讲一下在操作时遇到的坑,以及自己对Ingress的理解.

## 为什么要使用Ingress

先说一下为什么要安装Ingress吧,Ingress英文翻译为入口,在k8s集群中也不难理解就是外部流量的入口,k8S作为一个容器的管理工具,部署在上面的服务很多都需要和外部进行交互,有的是提供前段页面的,有的是提供接口调用的,k8s在管理Deployment时使用了Service作为代理,而在处理整个集群中服务的调用负载用的就是Ingress.

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g8qolvngotj305207imxb.jpg)

上图是流量从LB到k8s上对应pod的一个过程,实际上虽然ingress的意思是入口但是他对外的暴露依然需要一个service,由此我们不难猜到ingress在k8s中也是作为一个pod存在的.

## 配置Ingress

既然是作为一个pod存在的那自然需要我们自己动手进行部署.

其实k8s提出Ingress的概念是为了抽象下游代理服务的具体实现,目前对于Ingress Controller的实现还是比较多的,我们熟知的**Nginx**以及后起之秀**traefik**.

> 这里不对rraefik做过多的介绍,大家可以通过[官方文档](https://traefik.io/)自行查阅.
>
> 这里有一篇[文章](https://stackshare.io/stackups/nginx-vs-traefik)是关于Nginx与Traefik的对比

既然Ingress Controller代表着负载均衡器,那路由的规则怎么配置呢?这就需要Ingress了,在k8s中ingress是作为一个对象存在的.

```yaml
  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    name: test-ingress
  spec:
    rules:
    - host: nginx-test.zhengyiwoaini.top
      http:
        paths:
        - backend:
            serviceName: nginx
            servicePort: 80
    tls:
    - hosts:
      - nginx-test.zhengyiwoaini.top
      secretName: https-test
```

Ingress使用rule进行规则配置,这里配置的规则交给master之后会翻译成Ingress Controller所需要的配置文件(对于Nginx而言就是nginx.config)

```yaml
server {
  # 监听http
	listen 80;
	# 监听https
	listen 443 ssl;
	ssl_certificate /etc/nginx/secrets/default-https-test;
	ssl_certificate_key /etc/nginx/secrets/default-https-test;
  # 监听的域名
	server_name nginx-test.zhengyiwoaini.top;
  # 转发的端口
	location / {
		proxy_pass http://default-test-ingress-nginx-test.zhengyiwoaini.top-nginx-80;
	}
}
```

可以对比上面的两个文件,一个是ingress另个一是nginx.config熟悉nginx配置的小伙伴不难发现,Ingress Controller实际上就是监听Ingress中Rules的变化然后重放到Nginx上.

## 添加TLS认证

Tt





## 参考文档

- [nginx-ingress-controller官方文档](https://github.com/nginxinc/kubernetes-ingress/blob/master/docs/installation.md)