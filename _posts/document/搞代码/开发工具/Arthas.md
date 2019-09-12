## 写在前面
Arthas是阿里爸爸开源出来的一款java监控工具,命令行操作,完全面向开发人员,其专业行令人发指.

## 安装过程
```
wget https://alibaba.github.io/arthas/arthas-boot.jar
java -jar arthas-boot.jar
```
下载完jar包之后需要联网下载相关的jar包,当然这些操作都是arthas-boot.jar自动执行的

## 相关命令介绍
由于官方文档给的已经很全了我这里就工作中比较常用的几个命令描述下

```text
 NAME         DESCRIPTION
 help         Display Arthas Help
 keymap       Display all the available keymap for the specified connection.
 sc           Search all the classes loaded by JVM
 sm           Search the method of classes loaded by JVM
 classloader  Show classloader info
 jad          Decompile class
 getstatic    Show the static field of a class
 monitor      Monitor method execution statistics, e.g. total/success/failure count, average rt, fail rate, etc.
 stack        Display the stack trace for the specified class and method
 thread       Display thread info, thread stack
 trace        Trace the execution time of specified method invocation.
 watch        Display the input/output parameter, return object, and thrown exception of specified method invocation
 tt           Time Tunnel
 jvm          Display the target JVM information
 ognl         Execute ognl expression.
 dashboard    Overview of target jvm's thread, memory, gc, vm, tomcat info.
 dump         Dump class byte array from JVM
 options      View and change various Arthas options
 cls          Clear the screen
 reset        Reset all the enhanced classes
 version      Display Arthas version
 shutdown     Shutdown Arthas server and exit the console
 session      Display current session information
 sysprop      Display, and change the system properties.
 sysenv       Display the system env.
 redefine     Redefine classes. @see Instrumentation#redefineClasses(ClassDefinition...)
 history      Display command history
```
![asd](https://alibaba.github.io/arthas/en/_images/dashboard.png)

## 官方文档
官方文档真的已经很想详细了(阿里爸爸很良心)
- [中文文档地址](https://alibaba.github.io/arthas/index.html)
- [英文文档地址](https://alibaba.github.io/arthas/en/index.html)