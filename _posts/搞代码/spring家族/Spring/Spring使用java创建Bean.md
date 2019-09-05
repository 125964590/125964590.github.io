# 概述
**众所周知,Spring框架是空值反转(IOC)或依赖注入(DI)弄湿的推动因素,这种推动是通过容器的配置实现的,过去,Spring允许开发人员使用xnl配置,通过利用应用程序上下文XML文件来管理bean依赖性,此文件处于应用程序的外部,包含bean以及该应用程序的依赖项定义.这种方法称为基于java的配置,不同于xml基于java的配置能购是您已变成的思想管理bean.这可通过运用多种注释实现.这篇文章将演示java配置示例,并将其与传统的xml配置方法相比.本文将按照如下步骤演示基于java的配置的基本用法:**
- 理解@Configuration和@Bean注释
- 使用AnnotationConfigApplicationContext注册配置类
- 配置web应用程序
- 实现bean生命周期回调和范围

## 理解@Configuration和@Bean注释
在理想场景中,您可以在表示应用程序上下文的xml重定义bean.以下代码展现了创建课程用力中的上到下文xml以及bean
清单1.xml与bean定义
```xml
<beans>
    <bean id="course" class="demo.Course">
        <property name="module" ref="module"/>
    </bean>
     
    <bean id="module" class="demo.Module">
        <property name="assignment" ref="assignment"/>
    </bean>
     
    <bean id="assignment" class="demo.Assignment" />
</beans>
```
以上 XML 就是您在使用 Spring 配置 bean 时通常会编写的代码。这段 XML 代码定义了 Course bean，它引用 Module bean。Module bean 有一个 Assignment bean 的引用。您现在要删除这段 XML，编写同等效果的 Java 代码。您将使用基于 Java 的配置定义上面指定的 bean。我们会将 XML 替换为 Java 类，这个 Java 类现在将用作 bean 配置的平台。我们将这个类命名为 AppContext.java。以下代码展示了 AppContext 类。
```java
@Configuration
public class AppContext {
    @Bean
    public Course course() {
        Course course = new Course();
        course.setModule(module());
        return course;
    }
 
    @Bean
    public Module module() {
        Module module = new Module();
        module.setAssignment(assignment());
        return module;
    }
 
    @Bean
    public Assignment assignment() {
        return new Assignment();
    }
}
```
正如您通过以上代码所看到的那样，现在可以以编程的方式将 bean 定义为基于 Java 的配置的一部分。AppContext 类现在就像 XML 一样表示配置类。这是通过利用 @Configuration 注释实现的。@Configuration 注释位于类的顶端。它告知 Spring 容器这个类是一个拥有 bean 定义和依赖项的配置类。@Bean 注释用于定义 bean。上述注释位于实例化 bean 并设置依赖项的方法上方。方法名称与 bean id 或默认名称相同。该方法的返回类型是向 Spring 应用程序上下文注册的 bean。您可使用 bean 的 setter 方法来设置依赖项，容器将调用它们来连接相关项。基于 Java 的配置也被视为基于注释的配置。
## 使用AnnotationConfigApplicationContext注册配置类
在传统 XML 方法中，您可使用 ClassPathXmlApplicationContext 类来加载外部 XML 上下文文件。但在使用基于 Java 的配置时，有一个 AnnotationConfigApplicationContext 类。AnnotationConfigApplicationContext 类是 ApplicationContext 接口的一个实现，使您能够注册所注释的配置类。此处的配置类是使用 @Configuration 注释声明的 AppContext。在注册了所述类之后，@Bean 注释的方法返回的所有 bean 类型也会得到注册。以下代码演示了 AnnotationConfigApplicationContext 类的使用：

清单 3. 使用 AnnotationConfigApplicationContext 注册 AppContext 类
```java
public static void main(String[] args) {
  ApplicationContext ctx = new AnnotationConfigApplicationContext(AppContext.class);
  Course course = ctx.getBean(Course.class);
  course.getName();
}
```