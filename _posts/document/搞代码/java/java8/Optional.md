# Optional的学习与实战
整片文章大部分内容来自java8实战这本书,我在这里也是将自己的学习过程记录下来,并且整理成笔记给需要的人提供一个方便,在学习的过程中主要有以下几点疑惑:
- 不明白Optional的作用是什么?
- 不清楚在什么情况下使用Optional?
- 面对多层嵌套的情况Optional如何拆解?
- Optional和Stream之间的关系?

## 写在前面
作为一个java开发程序猿如果说你没有遇到过空指针的问题,那真的是太神奇了,在我们的工作中,常常会想尽一切办法避免空指针的.事实上也有人曾经提出空引用的想法,就是对null依然进行引用,但是这种做法所带来的可怕影响就是会在后期带啦大量的NullPrinterException异常.
本文的目的是使用Optional来取代null,并且使用Optional来去除代码中大量的null检查从而提高我们程序的可读性和代码的健壮性.

### Null带来的种种问题
- 错误之源:NullPointExpection是java开发中最经常遭遇的异常.
- 是你的代码膨胀:页面中充斥着过多而null检查,是代码的可读性大大降低.
- 自身毫无意义:自身没有任何意义,这本身就是一种错误的建模.
- 破坏了java的哲学性:java一直试图让人们避免接触指针的问题,但是null是唯一的例外
- 在java类型系统上开了一个口子:null并不属于任何类型,这意味着他可以赋值给任意变量.

### 其他语言中null的替代品
在Groovy中可以通过安全导航操作符(Safe Navigation Operator,标记为?)
```groovy
carInsuranceName= person?.car?.insurance?.name
```
在使用变量是可能出现null的情况,但是在这种情况下不会抛出空指针的异常,而是将null沿着调用链一直传递下去.最终返回一个null.
在java8中引入了Optional的概念设置了一个新的类java.util.Option<T>的类型,从而对可能出现null的对象进行建模

## Optional类入门
如果我们需要使用一个对象,这个对象中可能存在null,我们可以声明其为Optional<Car>类型,变量存在是返回的是封装的对象,当变量不存在使返回的是一个空的Optional对象.在语法上你可以把这个当做一回事,但是实际中他们之间存才非常大的差别.如果你引用的是一个null,一定会触发NullPointException
,如果使用Optional.empty()就完全没有事.
**使用Optional而不是null是一个非常重要而又实际的语义区别.
使用Optional对你的代码进行重构.
```java
public class EssayCompositeQuestionDot {

    /**
     * paper name
     */
    private String paperName;

    /**
     * paper id
     */
    private int paperId;

    /**
     * Check to see if the object highlighted
     * In this class 'flag' default 'true'
     */
    private boolean flag = true;

    private List<Optional<StemList>> stemList = new LinkedList<>();

    private List<Optional<MaterialList>> materialList = new LinkedList<>();

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public class MaterialList {

        /**
         * material id
         */
        private Integer id;

        /**
         * material content
         */
        private String content;

        /**
         * Check to see if the object highlighted
         */
        private boolean flag = false;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public class StemList {
        /**
         * stem baseId
         */
        private Integer baseId;

        /**
         * stem detailId
         */
        private Integer detailId;

        /**
         * stem content
         */
        private String content;

        /**
         * Check to see if the object highlighted
         */
        private boolean flag = false;

    }
```

### 创建Optioanl对象
Optional提供了集中静态方法供我们直接使用
#### 声明一个空的Optional
直接调用静态工厂的方法Optional.empty创建一个对象
```java
    Optional<EssayQuestionGroupResponse> empty = Optional.empty();
```

#### 根据非空值创建Optional
直接调用讲台方法Optional.of,依据一个非空值创建一个Optional对象
```java
    Optional<EssayQuestionGroupResponse> essayQuestionGroupResponse = Optional.of(new EssayQuestionGroupResponse());
```

#### 可接受null的Optional
使用讲台工厂方法Optional.ofNullable创建一个允许null值得Optional对象
```java
 EssayQuestionGroupResponse essayQuestionGroupResponseNull = new EssayQuestionGroupResponse();
        Optional<EssayQuestionGroupResponse> essayQuestionGroupResponseNullOptional = Optional.ofNullable(essayQuestionGroupResponseNull);
```

### 获取Optional中的对象
我们将对象放到了Optional中,他们本身提供了一个get方法,但是如果我们不按照约定进行get那么他依然会抛出一个NullException
```java
     EssayQuestionGroupResponse essayQuestionGroupResponseNull = new EssayQuestionGroupResponse();
        System.out.println(essayQuestionGroupResponse.map(EssayQuestionGroupResponse::getGroupId).get());
```
```java
Exception in thread "main" java.util.NoSuchElementException: No value present
	at java.util.Optional.get(Optional.java:135)
	at com.huatu.search.optional.OptionalGetTest.main(OptionalGetTest.java:19)
```

#### 使用map从optional对象中提取和转化值
从对象中提取信息这是很常见的模式,Optional提供了一个**map**方法,他的工作方式如下.
```java
essayQuestionGroupResponse.map(EssayQuestionGroupResponse::getGroupId);
```
同学们也很容易发现,如果想要获取嵌套对象,让然使用map是不可取的,这里需要使用**flatMap**
```java
essayQuestionGroupResponse
        .flatMap(EssayQuestionGroupResponse::getMaterialList)
        .map(EssayQuestionGroupResponse.Material::getMaterialContent).orElse("null"))
```
在最后使用orElse如果在调用链中的认识一环出现空指针则会返回orElse中的内容
```java
    Optional<EssayQuestionGroupResponse> essayQuestionGroupResponse = Optional.of(new EssayQuestionGroupResponse());
        //这里会出现空值,以后在研究
        System.out.println(essayQuestionGroupResponse
                .flatMap(EssayQuestionGroupResponse::getMaterialList)
                .map(Material::getMaterialContent).orElse("null"));
    }
```

### 默认行为以及街引用Optional对象
再生产中有如下几种方式获取到Optional中的对象
- get()是最简单的方法,并且不安全,如果变量存在直接返回封装的变量值,否做就跑出一个NoSuchElementException异常 .
- orElse(T other)当Optional中不包含值得时候提供一个默认值.
- orElseGet(Supplier<? extends T>)是上一个方法的延时调用版,需要提高性能可以使用此方法.
- orElseThrow(Supplier<? extends exceptionSupplier>)如果对象为空的时候回抛出一个异常.
- ifPresent(Consumer<? super T>) 让变量在存在的时候作为一个消费者传入一个方法否则不执行任何操作.

## 实战实例
有效的使用Optional类意味着在需要处理潜在缺失值得时候可以进行全面的反思了.

### 用Optional封装可能为null的值
现存的很多方法都是通过返回一个null来表示没有这个值,就好比我们经常使用的map.get()方法,如果该map中不存在对应属性则会返回一个null.这是用Optional将代码封装起来就可以减少if-else的判断.
```java
Optional<String> value=Optional.ofNullable(map.get("key"));
```
每次在获取潜为null的对象时都可以调用这个方法.

### 异常与Optional的对比
由于某种原因,函数无法返回某个值,这是除了返回null,JavaAPI通常的做法是抛出一个异常.这种情况比较典型的例子就是Integer.parseInt(String),如果转化失败则会抛出一个NumberFormatException,这种情况和之前的null相比唯一不同的处理方法就是使用try/catch,而之前是使用if/else进行判断.这时候我们可以用如下的方式完成这个功能.
```java
public static Optional<Integer> stringToInt (String s) {
    try {
        return Optional.of(Integer.parseInt(s);
    } catch (NumberFormatException e) {
        return Optional.empty();
    }
}
```
我们可以将多中类似的方法封装到一个工具类中进行使用.



## 总结
整片文章大部分内容来自java8实战这本书,我在这里也是将自己的学习过程记录下来,并且整理成笔记给需要的人提供一个方便,在学习的过程中主要有以下几点疑惑:
- 不明白Optional的作用是什么?
- 不清楚在什么情况下使用Optional?
- 面对多层嵌套的情况Optional如何拆解?
- Optional和Stream之间的关系?

通过系统的学习以上几点疑惑基本都已经解决了,在今后的开发中也将开始大量使用Optional这个方法来帮助自己的代码提高安全性和可读性.最后再附上一张思维导图用来描述Optional的总体概况

## 思维导图
![思维导图](https://ws1.sinaimg.cn/large/006rYg5Lly1fya27a5c1lj31oa0pb44o.jpg)