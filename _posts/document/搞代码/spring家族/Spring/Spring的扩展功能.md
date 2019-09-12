[TOC]
# Spring的扩展功能
Spring的核心就是BeanFactory,在原生的BeanFactory实现->XmlBeanFactory中,最小化的创建了一个BeanFactory,但是我们在Spring框架中使用的往往不是这个BeanFactory的实现.

## ApplicationContext

![Context的继承树](http://ww2.sinaimg.cn/large/006tNc79ly1g4k41u0vdoj30ip084aav.jpg)

通过查看这个继承树我们不难看出,在Spring2.0-Reactive的实现也是继承了ApplicationContext

![起点](http://ww1.sinaimg.cn/large/006tNc79ly1g4k46rsbxdj30jz0lzn0q.jpg)

上面这段代码来自SpringBoot的main函数,我们可以发现,refresh就是在这个地方调用的,Spring的整个留着也是从这里开始加载的

```java
	@Override
	public final void refresh() throws BeansException, IllegalStateException {
		try {
			super.refresh();
		}
		catch (RuntimeException ex) {
			stopAndReleaseReactiveWebServer();
			throw ex;
		}
	}
```
跟进去之后可以看到实际上是调用了AbstractApplicationContext的方法.
```java
@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				// 这里实现了一个空函数方便我们扩展
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				initMessageSource();

				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				onRefresh();

				// Check for listener beans and register them.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
			}
		}
	}
```
这里就是Spring 初始化的整个流程,获取到beanFactory之后就是开始对Spring功能的扩展,在这些扩展里给开发者流出了很多可扩展的切面,根据这里的,比如BeanFactoryPostProcess.

----------
**强调一下,Spring的扩展功能有很多,这里我只记录自己遇到的和使用过的**
----------

#### 添加ApplicationContextAwareProcess处理器
该处理器会扫描我们的Bean如果属于Aware会将我们需要的对象set进去.
实际上ApplicationContextAwareProcess实现了BeanPostProcess

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4k5vl4g1ij30gh03c3yr.jpg)

对于实现了BeanPostProcess的方法我们只需要关心他的两个方法:
- postProcessBeforeInitialization
- postProcessAfterInitialization
> 这里简单介绍下BeanPostProcess是如何实现的:
> ![](http://ww1.sinaimg.cn/large/006tNc79ly1g4ki68a21fj30m307uwf5.jpg)
> 这段代码的出处是创建Bean的时候,其内部会允许子类去实例化一个bean,所以这里会判断如果返回的bean不为空则直接跳出将bean返回
> ![](http://ww4.sinaimg.cn/large/006tNc79ly1g4ki8rswivj30m40b9wfw.jpg)

直接进入AwareProcess的前置处理器

![](http://ww2.sinaimg.cn/large/006tNc79ly1g550do5deaj30pq04wwf1.jpg)

这里可以看出如果我们的bean继承了这几个方法就会在这里将实现完成

由于我们的调用在BeanFactoryProcess中已经完成了,所以需要在接下来的实例化过程中将这些类忽略掉

![](http://ww2.sinaimg.cn/large/006tNc79ly1g550fnxeidj30me09rmzi.jpg)

该扩展为硬编码的形式存在,所以这个扩展点主要是给我们提供一些帮助(在创建bean的时候),这些参数的获取是通过实现传递的,所以需要在实例化bean的时候将他们忽略掉.

### FactoryBean的使用
通过xml的方式实现过于复杂,使用FactoryBean来实现接口,通过实现FactoryBean来实现该接口定制化实例化的bean逻辑.
通过调用getObject()方法来实现Bean的实例化

## BeanFactory的后处理
BeanFactory作为Spring容器功能的基础,存放着所有已加载的bean.我们在真正实例化bean之前可以在这里对其进行扩展.

### 调用BeanFactoryPostProcess

![](http://ww3.sinaimg.cn/large/006tNc79ly1g5591z4runj311y0o2aec.jpg)

调用BeanFactoryPostProcess是Spring通过硬编码的形式放在初始化流程中的,通过观察我们可以发现

下面这段代码逻辑出自invokeBeanFactoryPostProcessors(),这里就是调用BeanFactoryPostProcessors的地方,观看下面这个循环判断可以了解到为什BeanDefinitionRegistryPostProcessor中的postProcessBeanDefinitionRegistry会优先调用.
![](http://ww3.sinaimg.cn/large/006tNc79ly1g558lzbu3zj30j80750tj.jpg)

所有的BeanFactoryPostProcessor和BeanDefinitionRegistryPostProcessor都放在下面的这两个缓存中等待调用.

```java
	List<BeanFactoryPostProcessor> regularPostProcessors = new ArrayList<>();
	List<BeanDefinitionRegistryPostProcessor> registryProcessors = new ArrayList<>();
```

### 注册BeanPostProcessor
注册BeanPostProcessor的逻辑和上面的类似,对其进行一系列的校验之后将其存储到BeanFactory的缓存中

![](http://ww3.sinaimg.cn/large/006tNc79ly1g55afhk99uj30mj01ygln.jpg)


### 调用顺序
这里强调一下BeanFactoryPostProcessor和BeanDefinitionRegistry以及BeanPostProcessor的调用顺序

![](http://ww4.sinaimg.cn/large/006tNc79ly1g55an0vekbj31210dd41i.jpg)

#### BeanDefinitionRegistry
BeanDefinitionRegistry是最先调用的,调用的位置在上文已经给出,是初始化BeanFactoryPostProcessor的时候进行调用,而且调用的时候没有先后顺序,只是对BeanFactoryPostProcessor本身进行加工

声明了几个就调用几次

#### BeanFactoryPostProcessor
BeanFactoryPostProcessor的调用位置会有以下之中:
- 按顺序调用
- 无需调用

首先调用排序好的BeanFactoryPostProcessor,在调用没有排序的BeanFactoryPostProcessor

声明了几个就调用几次

#### BeanFactoryPostProcessor
BeanFactoryPostProcessor这个方法的调用位置根据Bean的实例化决定,在Bean的实例化前后会调用其中的两个方法

所有的Bean在实例化的时候都会调用

### 初始化ApplicationEventMulticaster
ApplicationEventMulticaster的初始化过程比较简单,这里在默认的情况下会注册一个Spring默认的管理器,如果用户自己指定也可以由用户注入.

这段代码的源码内容也比较简单直观,这里把他的多播器列出来

```java
	@Override
	public void multicastEvent(final ApplicationEvent event, ResolvableType eventType) {
		ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
		for (final ApplicationListener<?> listener : getApplicationListeners(event, type)) {
			Executor executor = getTaskExecutor();
			if (executor != null) {
				executor.execute(new Runnable() {
					@Override
					public void run() {
						invokeListener(listener, event);
					}
				});
			}
			else {
				invokeListener(listener, event);
			}
		}
	}
```

这里会将所有的时间广播到所有的监听器上,如果有异步执行器则会使用异步处理.

### 注册监听器

这里顾名思义,就是将我们的listener注册到监听器上,简而言之就是add()到缓存的List中.

## 初始化非延迟加载单例
在这里会将所有的非延迟单例完成加载,这么做是一件好事,因为我们的bean错误可以及时发现.

## finishRefresh
Spring的开启和结束事件就是在这里发出的,通过ApplicationEventMulticaster来发布Spring的各种状态.