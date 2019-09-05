# Bean的实例化

经过xml和注解的解析,我们已经扫描的了所有的Bean,下面开始讲Bean创建出来.

ApplicationContext里面有一个方法是getBean(),那么就跟着这个方法,来看看Bean是如何获取的.

## 从缓存中获取
同一个Bean容器只会创建一次,多次获取会直接从缓存中加载.

这里面有一个细节需要注意,Spring为了解决循环依赖,在创建Bean的时候回先将ObjectFactory(Bean的实例创建方法)曝光加入到缓存中,一旦某一个bean在创建的时候需要依赖另一个bean,则直接使用ObjectFactory.

![从缓存中获取](http://ww2.sinaimg.cn/large/006tNc79ly1g4ljkd9xwej30md0bg763.jpg)

通过上图可以看出,在创建bean的时候,会从三个地方检察缓存,如下图所示:

![缓存对象](http://ww3.sinaimg.cn/large/006tNc79ly1g4ljq5q7z9j30n205yt9n.jpg)
根据bean name 去Map中获取相应的缓存,**强调!!这个缓存获取是有顺序的**

当==singletonObjects==和==earlySingletonObjects==中都没有获取到缓存的时候会使用ObjectFactory去创建一个bean实例,并且将bean方到缓存中**强调!!需要保证==singletonObjects==和==earlysingletonObjects==互斥.

==singletonObjects==和==earlysingletonObjects==的不同之处在于,当一个单例的bean被放到这里后,当bean还在创建过程中,就可以通过getBean方法获取到了.

## 从bean的实例中获取对象
在getBean方法中,getObjectForBeanInstance是一个非常高频的方法,无论是从缓存中还是根据不同的scope策略加载的bean,总之我们得到这个方法后要做的第一件事就是检查一下这个方法的准确性,其实就是判断这个bean是否是FactoryBean类型的bean,如果是则直接调用getObject()作为返回值.[^01]

跟进方法我们可以看到如下的代码

![获取bean](http://ww3.sinaimg.cn/large/006tNc79ly1g4lmfgtyknj30pr06ugm9.jpg)

在这里展示了如何获取bean,==doGetObjectFromFactoryBean==这个方法针对factory进行处理,实际底层就是调用了getObject()这个方法

图中==postProcessObjectFromFactoryBean==这个方法是对bean的后置加工

我们此时需要明确一点,在创建bean的过程中,Spring最大程度的保证,所有的bean都会调用BeanPostProcess所实现的两个方法
- postProcessBeforeInitialization
- postProcessAfterInitialization
这两个方法可以基于我们针对自己的业务进行扩展

## 获取单例
之前所讲的,都是从缓存中获取单例的过程,如果缓存中不存在bean就要从头开始加载了,Sping中的getSingleton吃的重载方法实现了bean的加载过程.

这个方法的调用实际上是通过回调的形式实现的

![获取单例](http://ww1.sinaimg.cn/large/006tNc79ly1g4lmww26saj30ep043aa8.jpg)

真正创建ObjectFactory的方法试下createBean中实现的

## 准备创建bean
进入createBean中可以看到真正获取bean的逻辑在doCreateBean中(这也符合Spring的代码风格,真正做事情的是放在do开头的方法中)

![真正创建bean的方法](http://ww3.sinaimg.cn/large/006tNc79ly1g4ln0fqjhej30j604xt91.jpg)

下面这张图是在进入doCreateBean方法之前,调用了==resolveBeforeInstantiation==这个方法

![doCreate](http://ww3.sinaimg.cn/large/006tNc79ly1g4ln264a73j30lz04naaa.jpg)

进入其内部发现,实际上是判断是否通过==BeanPostProcessor的方法获取bean的实例,**这里也是提供给我们的扩展点**

![进入](http://ww3.sinaimg.cn/large/006tNc79ly1g4ln43cs2wj30nk07iq3q.jpg)


## 实例化Bean
实例化bean的过程非常复杂,这里我们想一想也能知道为什么:
1. 如果是单例则先清空缓存
2. 实例化bean,将BeanDefinition转换为BeanWrapper
    - 如果存在工厂方法则通过工厂方法进行创建
    - 一个类有多个构造函数,需要根据参数去锁定使用的构造函数
    - 如果没有工厂方法,也没有带参数的构造函数,那么默认使用无参构造函数进行创建
3. 依赖处理:
    - 在Spring中会存在循环依赖的问题,如过是一个单例Bean,例如A和B循环依赖,当A需要注入B中的属性是,不是再去创建B,而是通过放入缓存中的ObjectFactory来创建
4. 属性注入,在这里将bean所需要的全部属性注入
5. 循环依赖检查,如果有循环依赖在这里直接抛出异常
6. 注册DisposableBean,如果bean销毁的时候直接调用方法.
7. 完成创建


### CreateBeanInstance

![create](http://ww4.sinaimg.cn/large/006tNc79ly1g4lnqhkpb9j30r608dwfx.jpg)




[^01]: Spring源码深度解析2