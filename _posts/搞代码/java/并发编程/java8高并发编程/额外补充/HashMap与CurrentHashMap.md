# HashMap与ConcurrentHashMap
![](https://ws3.sinaimg.cn/large/006tKfTcly1g0jt2dr5rgj31ft0u0ncv.jpg)
- 容量:内部空间大小(默认1<<4,最大1<<30)
- 加载因子:每次扩容的阈值(默认0.75)
- 从java8之后长度超过8之后自动转化为红黑树

## rehash
### 单线程环境
![](https://ws2.sinaimg.cn/large/006tKfTcly1g0jt53agb7j31dy0u01kx.jpg)


### 多线程环境
![](https://ws2.sinaimg.cn/large/006tKfTcly1g0jt5k3o9oj31c80u07pb.jpg)

![](https://ws2.sinaimg.cn/large/006tKfTcly1g0jta4knv1j31ke0tiqge.jpg)

![](https://ws1.sinaimg.cn/large/006tKfTcly1g0jtabpblsj31js0tc4bt.jpg)

## ConcurrentHashMap
![](https://ws2.sinaimg.cn/large/006tKfTcly1g0jtjgdhfkj31bs0u0tvy.jpg)

![](https://ws4.sinaimg.cn/large/006tKfTcly1g0jtldhv94j31c80u0n7d.jpg)