# jumpserver

jumpserver是一个跳板机,说简单点就是一个帮助我们管理服务器的东西,我们可以通过跳板机将资源暴露在外网上,也可以使用跳板机来管理多用户,对于协作开发有很大的帮助.

## 官方文档带给我们的帮助
jumpserver是中国开源的一个产品,他的官方文档对于我们来说非常的友好,这里我主要提一下如何阅读官方文档以及官方文档的中比较重要的部分

## Docker安装

由于是个人使用我安装jumpserver的方式是通过docker,这里没有什么特殊的参数

```shell
# 生成随机加密秘钥, 勿外泄
$ if [ "$SECRET_KEY" = "" ]; then SECRET_KEY=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 50`; echo "SECRET_KEY=$SECRET_KEY" >> ~/.bashrc; echo $SECRET_KEY; else echo $SECRET_KEY; fi
$ if [ "$BOOTSTRAP_TOKEN" = "" ]; then BOOTSTRAP_TOKEN=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 16`; echo "BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN" >> ~/.bashrc; echo $BOOTSTRAP_TOKEN; else echo $BOOTSTRAP_TOKEN; fi

$ docker run --name jms_all -d -p 18000:80 -p 2222:2222 -e SECRET_KEY=$SECRET_KEY -e BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN jumpserver/jms_all:1.4.8
```

## 使用说明

 jumpserver具体的使用可以分为一下几个模块

- 用户管理
- 资产管理
- 会话管理

在使用的时候这几块内容需要详细阅读

## 相关文档

- [官方文档](https://docs.jumpserver.org/zh/master/)