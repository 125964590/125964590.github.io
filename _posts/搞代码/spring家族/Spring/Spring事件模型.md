## 事件驱动模型简介
事件驱动模型也就是我们常说的观察者,或者发布-订阅模型;理解他的几个关键点:
- 通过对象间的一对多关系实现松耦合(多方);
- 当目标方发送改变(发布),观察者(订阅者)就可以接收到改变;
- 观察者如何处理目标无权干涉,实现松耦合

### 举例
在业务上需要将复杂的事件解耦,通过异步去处理
![spring事件模型](http://dl2.iteye.com/upload/attachment/0086/6190/a4f53a3c-bc09-3217-ad67-97e61dc6b8bc.png)

增加了一个Listener来解耦UserService和其他服务,即注册成功后,只需要通知相关的监听器,就不需要关系他们如何处理.增删功能非常容易.

这是一个典型的事件处理模型/观察者,**解耦目标对象和他的依赖对象,目标只需要通知他的以来对象,具体怎么处理以来对象自己决定**.比如同步还是异步,低延迟还是非延迟等.

同时也符合DIP(依赖倒转原则),依赖与抽象,而不是具体.  

还需要记住一个原则**接口的目的是抽象,抽象类的目的是复用**

### 模型规范

#### Java模型
JavaBean规范提供了JavaBean的PropertyEditorSupport以及PropertyChangeListener

同事java还提供了相应的模型接口Observable,以及Observer

#### Spring事件模型
spring的事件驱动模型体系图:
![体系图](http://dl2.iteye.com/upload/attachment/0086/6530/2c76c311-074b-3460-87a0-b9ffc8e89f88.png)

##### 事件
具体代表者是:ApplicationEvent
- 继承自JDK的EventObject,JDK要求所有的时间将继承它,并通过source得到事件源;
- Spring系统默认提供了如下事件
![事件](http://dl2.iteye.com/upload/attachment/0086/6532/8637476c-9877-3432-ae82-e85745f27cc3.png)