# 使用maven编译使项目中存在的jar包变大

## 错误原因
在项目发布的过程中提示
```
messing jar /opt/jar/elastic-apm-agent-1.4.0.jar 
```
实际上是因为,我们在打包项目的时候将这个jar一起放到了docker中,然后进行运行.


## 纠错过程

1. 首先怀疑在构建镜像的时候没有将jar包打进去,但是经过测试jar包确实打进去了.
2. 一开始jar包是放在/tmp目录的怀疑tmp目录下jar包不生效<div style="color:red">错误做法:首先不说tmp到底会不会有问题,单从之前也放在tmp目录下就不应该怀疑这个</div>
3. 怀疑java -jar命令在docker 中出现问题,或者是docker的基础镜像有问题<div style="color:red">错误做法:和上一条类似,这些都是没有改变的元素没有必要怀疑</div>
4. 怀疑是jar包本身除了问题,对镜像内部的jar和外部的进行md5.

在第四步的时候已经大概定位到问题了,在几个转换jar包的位置进行md5校验最终发现是maven在构建的时候将jar包更改

## 解决方法
1. 尝试在maven构建的时候排除==elastic-apm-agent-1.4.0.jar==的编译
2. 再通过maven其他命令将==elastic-apm-agent-1.4.0.jar==原样copy到target目录下进行docker构建

## 反思
反思整体bug解决流程实际上定位问题的过程不是特别复杂,但是问题诡异在maven对==elastic-apm-agent-1.4.0.jar==进行了更改该,所以导致排除问题的时候多了一些错误的方向.

<div style="color:red">实际上在定位问题的,时候只要思路正确做一些大胆的猜想也是可以的</div>