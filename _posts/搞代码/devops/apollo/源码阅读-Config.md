# 源码阅读---Config

![](https://ws1.sinaimg.cn/large/006tNc79ly1g24tocwl4zj30bn08u0su.jpg)

其中Config是所有类的父类定义了公共接口这里不做赘述直接看子类实现

### AbstractConfig
在Config重定义了很多接口交给==AbstractConfig==去实现其内部包括
- 共享的线程池==m_executorService==
- 监听器的集合(包括对监听器的添加和移除)
- 各种缓存
- 实现了所有获取属性值的方法

### DefaultConfig
其中包含了对==AbstractConfig==的一些扩展,也是在工作中真正拿来使用的,它本身还实现了==ReositoryChangeListener==方法,因为在==DefaultConfig==它本身也需要接受仓库的改变.