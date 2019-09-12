## 写在前面
先说一下目前遇到的情况:
1. 再生产中使用了Apollo;
2. 在我们的环境中使用Apollo的方式是直接改写他的源码,然后本地部署,将他的Client进行打包拿到项目中进行使用(基本不会改变Apollo的源码);
3. 由于公司内部很多组件使用Start的方式进行编写,需要了解Apollo对配置文件的加载过程;
4. 由于公司Apollo的使用比较混乱,导致一个项目不同的环境有多个branch的存在;
5. 阅读Client的工作原理,并可以动态的修改NameSpace.

## Client的包结构
![包结构](https://ws2.sinaimg.cn/large/006tNc79ly1g1xv2cgalvj308r0blmxj.jpg)

Apollo的包结构非常的整洁,从上到下分大概是:
1. 一些实体(先不要去管他们就好);
2. 异常处理(封装了异常处理,没啥卵用);
3. 内部的处理类(这个比较重要,很多基本所有ApolloClient所做的是都和他有关);
4. 一些工厂方法;
5. Spring相关的(spring配置比不可少的,我们主要关心的地方);
6. 一些工具类

![spring启动位置](https://ws4.sinaimg.cn/large/006tNc79ly1g1xva3ewosj305b03l748.jpg)

通过观察resources发现内部包含==spring.factories==,进去看看就可以找到Client的启动入口

## 项目入口
![入口](https://ws4.sinaimg.cn/large/006tNc79ly1g1xvcc72ywj30hf03q3yw.jpg)
通过观察我们发现这个类中分别声明了自动配置、环境初始化加载、环境后置加载三个引入点,他们的加载顺序是:
1. ApolloApplicationContextInitializer
2. ApolloAutoConfiguration
3. ApolloApplicationContextInitializer

我们对源码的阅读也随着他初始化的过程查看.

### 环境初始化(ApolloApplicationContextInitializer)
这个列继承了==ApplicationContextInitializer==(这还用说么(ノへ￣、))直接来看他的实现方法吧
```java
  @Override
  public void initialize(ConfigurableApplicationContext context) {
  
    //获取已经传入的配置
    ConfigurableEnvironment environment = context.getEnvironment();
    
    //初始化Apollo的系统参数,就是我们熟知的'app.id'等
    initializeSystemProperty(environment);
    
    //判断是否需要开始配置(针对于apollo.bootstart的需求)
    String enabled = environment.getProperty(PropertySourcesConstants.APOLLO_BOOTSTRAP_ENABLED, "false");
    if (!Boolean.valueOf(enabled)) {
      logger.debug("Apollo bootstrap config is not enabled for context {}, see property: ${{}}", context, PropertySourcesConstants.APOLLO_BOOTSTRAP_ENABLED);
      return;
    }
    logger.debug("Apollo bootstrap config is enabled for context {}", context);

    if (environment.getPropertySources().contains(PropertySourcesConstants.APOLLO_BOOTSTRAP_PROPERTY_SOURCE_NAME)) {
      //already initialized
      return;
    }
    
    //解析配置文件获取namespaces
    String namespaces = environment.getProperty(PropertySourcesConstants.APOLLO_BOOTSTRAP_NAMESPACES, ConfigConsts.NAMESPACE_APPLICATION);
    logger.debug("Apollo bootstrap namespaces: {}", namespaces);
    List<String> namespaceList = NAMESPACE_SPLITTER.splitToList(namespaces);
    
    //创建一个容器来接收Apollo上的配置
    CompositePropertySource composite = new CompositePropertySource(PropertySourcesConstants.APOLLO_BOOTSTRAP_PROPERTY_SOURCE_NAME);
    for (String namespace : namespaceList) {
      Config config = ConfigService.getConfig(namespace);
      
      //@1@
      composite.addPropertySource(configPropertySourceFactory.getConfigPropertySource(namespace, config));
    }

    //这里讲解析的配置文件添加到First(这里说明下,由于Spring在解析配置文件的时候是前面的覆盖后见面的,所以为了让Apollo的配置文件优先级高于本地的配置文件,这里放在最前面)
    environment.getPropertySources().addFirst(composite);
  }
```
上文@1@出使用的是谷歌的工具类通过反射创建的一次性对象**这里需要去研究下为什么要使用反射来创建对象,非要秀操作么**

### 自动配置(ApolloAutoConfiguration)

```java
  private final ConfigPropertySourceFactory configPropertySourceFactory = SpringInjector
      .getInstance(ConfigPropertySourceFactory.class)
```

==ApolloAutoConfiguration==这个就是我们要找的类

```java
@Configuration
@ConditionalOnProperty(PropertySourcesConstants.APOLLO_BOOTSTRAP_ENABLED)
@ConditionalOnMissingBean(PropertySourcesProcessor.class)
public class ApolloAutoConfiguration {

  @Bean
  public ConfigPropertySourcesProcessor configPropertySourcesProcessor() {
    return new ConfigPropertySourcesProcessor();
  }
}
```
进入类的内部很明显是通过==BeanFactory==实现的对象托管.
```java
public class PropertySourcesProcessor implements BeanFactoryPostProcessor, EnvironmentAware, PriorityOrdered {
```
#### bean初始化
初始化过程和==ApolloAutoCnfiguration==中的流程基本一样,这里就不再一次重复了
```java
  //是的已经猜到了,最高优先级的加载顺序,因为需要通过Apollo去加载其他项目的依赖配置.
  @Override
  public int getOrder() {
    //make it as early as possible
    return Ordered.HIGHEST_PRECEDENCE;
  }
  //通过集成EnvironmentAware来为当前类获取环境配置对象
  @Override
  public void setEnvironment(Environment environment) {
    //it is safe enough to cast as all known environment is derived from ConfigurableEnvironment
    this.environment = (ConfigurableEnvironment) environment;
  }
```
初期的准备都已经完成了,但是我们观察==postProcessBeanFactory==可以发现,还有一个初始化自动配置的方法,在这里我们详细的看下这个方法
```java
  @Override
  public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
    if (INITIALIZED.compareAndSet(false, true)) {
      initializePropertySources();
      //初始化配置监听器
      initializeAutoUpdatePropertiesFeature(beanFactory);
    }
  }
```
进入方法进行查看
```java
  private void initializeAutoUpdatePropertiesFeature(ConfigurableListableBeanFactory beanFactory) {
    //判断是否开启配置注入
    if (!configUtil.isAutoUpdateInjectedSpringPropertiesEnabled()) {
      return;
    }

    //创建消息监听者
    AutoUpdateConfigChangeListener autoUpdateConfigChangeListener = new AutoUpdateConfigChangeListener(
        environment, beanFactory);

    //将监听者分给所有的配置资源管理者
    List<ConfigPropertySource> configPropertySources = configPropertySourceFactory.getAllConfigPropertySources();
    for (ConfigPropertySource configPropertySource : configPropertySources) {
      configPropertySource.addChangeListener(autoUpdateConfigChangeListener);
    }
  }
```

### 结语
读到这里我们已经大致的了解了Apollo在项目启动的时候会为我们做什么,针对Spring的项目主要使用的是BeanFactoryPostProcessor来进行配置拉取.并且注册了一个动态修改配置的.

```java
/**
 * @author jbzm
 * @date 2019-04-10 14:36
 */
@Configuration
public class ApolloCustomConfiguration
        implements BeanFactoryPostProcessor, EnvironmentAware, Ordered {

    private static final int SUPPRESSION_PRIORITY = 100;

    private ConfigurableEnvironment configurableEnvironment;

    private final ConfigPropertySourceFactory configPropertySourceFactory =
            SpringInjector.getInstance(ConfigPropertySourceFactory.class);

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)
            throws BeansException {
        ApolloBranch apolloBranch = new ApolloBranch();
        apolloBranch.setBranchName(configurableEnvironment.getProperty(APOLLO_BRANCH));
        Config config = ConfigService.getConfig(apolloBranch.getBranchName() + ".rw.jdbc");
        CompositePropertySource compositePropertySource =
                new CompositePropertySource(APOLLO_BRANCH);
        compositePropertySource.addPropertySource(
                configPropertySourceFactory.getConfigPropertySource(APOLLO_BRANCH, config));
        configurableEnvironment.getPropertySources().addFirst(compositePropertySource);
    }

    @Override
    public void setEnvironment(Environment environment) {
        this.configurableEnvironment = (ConfigurableEnvironment) environment;
    }

    @Override
    public int getOrder() {
        return SUPPRESSION_PRIORITY;
    }
}
```

