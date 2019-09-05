# Java内存区域与内存溢出异常

## 运行时数据区

在java运行时会将内存划分为以下若干个不同的数据区域,这里简称运行时数据区

![java虚拟机模型](http://static.zybuluo.com/Rico123/wozzd000rzpwwpz4eqi0xf1j/JVM%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B.png)

### 程序计数器

程序计数器(Program Counter Register)是一块很小的内存空间,**线程私有的**可以看做是当前线程所执行的字节码的行号指示器.在执行多线程任务时为了保证每个线程在切换的时候能回到之前的位置,每个线程都需要有一个独立的线程计数器.

### Java虚拟机栈

与程序计数器一样Java虚拟机栈(Java Virtual Machine Stacks)也是**线程私有**的,他的生命周期与线程相同,虚拟机栈描述的是java方法执行的内存模型:**每个方法在执行的同时都会创建一个栈帧(Stack Frame)用于存储局部变量表、操作数栈、动态链接、方法出口等信息.

在这个地方虚拟机会抛出两种异常

- StackOverflowError异常:当线程请求的站深度大于虚拟机所允许的深度就会抛出;
- OutOfMemoryError异常:如果虚拟机设置栈的深度可自动扩展,这是如果无法申请到足够的内存也会抛出异常.

### 本地方法栈

本地方法栈(Native Method Stack),与虚拟机栈非常相似,他们之间的区别就是虚拟机栈是给java方法提供服务,而本地方法栈是给Native方法服务.

本地方法栈所抛出的异常和java虚拟机栈所抛出的一样

### Java堆

对于大多数Java应用来说堆是内存管理中最大的一块.**是多个线程之间共享的**堆的唯一作用就是存放对象实例,几乎所有的对象实例都是在这里分配的.

Java堆是垃圾收集器主要处理的地方.现在的收集器基本都是基于分代收集算法,简单可以分成新生代和老生代.

如果堆中没有内存可以分配,并且堆无法进行扩展的时候将会抛出OutOfMemoryError.

### 方法区

方法区(Method Area)与Java堆一样,是各个线程共享的内存区域,用于存储已被虚拟机加载的类信息,常量,静态常量,这个区域中的数据相对稳定,但是也不是觉得的一直存在,虚拟机也会对方法区进行GC.

方法区同样会抛出OutOfMemoryError.

### 运行时常量池

Java虚拟机对运行时常量池(Runtime Constant Pool)没有详细的规范要求,运行时常量池可以动态的向其中添加数据/

当常量池无法申请内存的时候也会抛出OutOfMemoryError.

### 直接内存

直接内存(Direct Memory)并不是虚拟机运行时的一部分,也不是Java虚拟机规范定义中的内存区域,但是这部分仍然被频繁使用.其大小收到物理机本机内存的限制.

当直接内存无法申请内存空间是也会抛出OutOfMemoryError.

## HotSpot虚拟机对象探秘

### 对象的创建

虚拟机遇到一条new指令时,首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用,并且检查这个符号引用代表的类是否已经被加载、解析和初始化过.如果没有那就必须执行相应的类加载过程.

在类加载检查通过后,接下来虚拟机将为新生对象分配内存,对象所需内存大小在类加载完成后便可完全确定

### 对象的内存布局

在HotSpot虚拟机中,对象在内存中存储的布局分为三块:对象头(Header)、实例数据(Instance Data)和对齐填充(Padding).

##### 在头信息(Header)中包含两部分信息:

1. 第一部分用于存储对象自身运行时的数据.
2. 第二部分是类型指针,即对象指向他的类元数据的指针.

##### 实例数据部分

存放着父类以及子类的各种字段内容

##### 对其填充

由于HotSpot VM所管理的内存都是以8为倍数字节存储.

### 对象的访问定位

简历对象是为了适用对象,我们的Java程序需要通过栈上的reference数据来操作对的具体对象.由于reference类型在Java虚拟机规范中之规定了一个纸箱对象的引用,并没有定义这个引用应该通过何种方式去定位、访问队中的对象的具体位置，对象的访问方式也是取决于虚拟机的实现。主流的实现有两种，一种是通过句柄访问，一种是通过直接指针访问。

##### 句柄访问

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6e7wsrdh3j317g0mawi9.jpg)

##### 指针访问

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6e7xa98wgj317e0k00vf.jpg)

##### 两种访问方式各有优势

- 使用句柄的最大好处就是reference中存储的是稳定的句柄地址,在对象移动时只会改变句柄中实例数据指针,而reference本身不需要修改.
- 直接使用指针访问方式的最大好处就是速度更快,它节省了一次指针定位的时间开销.

## 实战:OutOfMemoryError异常

### Java堆溢出

```java
/**
 * VM options: -Xms20M -Xmx20M -XX:+HeapDumpOnOutOfMemoryError
 *
 * @author jbzm
 * @date 2019-08-27 14:56
 **/
public class HeapOOM {
    static class OOMobject {

    }

    public static void main(String[] args) {
        ArrayList<OOMobject> objects = new ArrayList<OOMobject>();
        while (true) {
            objects.add(new OOMobject());
        }
    }
}
```

o通过一段简单的代码实现OOM,这里可以看到是因为数组扩容申请新的空间导致超过Xmx的上限出现OOM,这种OOM属于内存溢出.

在查看OOM的时候我们要区分是内存溢出还是内存泄漏,如果是内存泄漏可以去判断GC Roots是否存在循环引用.

如果是因为内存溢出导致的OOM那我们就需要去查看Xmx是否还可以继续扩大,后者是从代码上去查看是否某些对象声明周期过长,持有状态时间过长等情况,尝试减少程序在运行期间的内存消耗.

### 虚拟机栈和本地方法栈溢出

在Java虚拟机规范中对栈描述了两种异常:

- 如果请求线程的站深度大于虚拟机所允许的最大深度,将抛出StackOutOfError异常.
- 如果虚拟机在扩展栈是无法申请到足够的存储空间,则抛出OutOfMemoryError异常.

```java
/**
 * VM options: -Xss128k
 *
 * @author jbzm
 * @date 2019-08-27 15:49
 **/
public class StackSOF {
    private int i = 1;

    private void recursive() {
        i = i++;
        recursive();
    }

    public static void main(String[] args) {
        StackSOF stackSOF = new StackSOF();
        stackSOF.recursive();
    }
}
```

### 方法区和运行时常量池溢出

由于运行时常量池是方法区的一部分,因此这两个区域的溢出属于同一种情况.在java1.7之后去除了永久带,也就是说对于方法区也会执行GC