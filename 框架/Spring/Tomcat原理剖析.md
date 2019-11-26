 2019.11.26
 
 * [Tomcat原理剖析](#tomcat原理剖析)
      * [Tomcat与Servlet](#tomcat与servlet)
      * [Socket](#socket)
      * [Tomcat运行示意图](#tomcat运行示意图)
      * [Tomcat的三个IO模型](#tomcat的三个io模型)
      * [Tomcat参数调优](#tomcat参数调优)
      * [JVM调优经验](#jvm调优经验)
      * [一个小坑哦](#一个小坑哦)

# Tomcat原理剖析
以前总是被告诉Tomcat是一个应用的容器，用来运行Servlet，但是自己仍然有很多疑惑，它和Apache等服务器又有什么区别，它是怎么做到的？今天终于能从源码的角度去分析tomcat，来了解其作为应用容器，具体是怎么实现的。

## Tomcat与Servlet
虽然我感觉目前的主流框架SpringBoot等已经淡化了Servlet的概念，不需要用户再去一个个从HttpServlerRequest这样的里面中获取参数再实现doGet等方法。只需要用一个Controller注解即可注入。但是Servlet仍是web系统中最重要的一环。是其来调用service层的服务，并向前端返回结果信息的。
而Tomcat是负责实例化并运行Servlet的一段应用程序，去看tomcat的目录，会发现在webapp下面,我们的项目中的WEB-INF文件，其中类都被编译好了，即可在其中运行。但是Tomcat是运行在端口中的，部署到服务器上之后，又如何从浏览器那里拿取需要的值并交给servlet呢？

## Socket
在Tomcat容器的最底层，是一层Socket，用来进行通信，socket是封装了TCP协议的一层消息通信，当数据以Http的形式到达Tomcat所在的服务器，其通过Socket从网卡中将数据读取过来并经过解析器（connector）将数据解析成一个`RequestFacade`的对象注入到servlet里面。也就有了`servletreques`t的各种属性，而用户需要的参数，请求也都在这些参数之中。值得注意的是，在这个解析器中，能配置很多重要的参数，最为重要的一个是Tomcat的IO模型。


## Tomcat运行示意图
这是我画的一张图来简单表示其运行的过程，可以看出其都是一层一层的封装，最后把数据传给了servlet里面。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126104811347.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)
## Tomcat的三个IO模型
Tomcat支持三种IO模型： BIO NIO APR

在我的8.5.8版本的conf目录下的server.xml配置文件中，可以看到

```
   <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    <!-- A "Connector" using the shared thread pool-->
```
可以看到，其使用的是默认的BIO的模型，为了优化性能，在并发量提升的情况下，可以通过
`protocol="org.apache.coyote.http11.Http11NioProtocol"` 指定使用NIO模型来接受HTTP请求。除此之外Tomcat还有apr模型。该模型是Apache HTTP服务器的支持库，处理静态资源效率有很大提高。

## Tomcat参数调优
* 更改IO模型，方法如上图
* 改变最大线程并发线程池的大小，一样可在server.xml配置，可以使得用户访问速度变快，但在大量并发的情况下线程满了可能阻塞

```
 <Executor name="tomcatThreadPool" namePrefix="catalina-exec-" 
maxThreads="1000" minSpareThreads="50" maxIdleTime="600000"/> 
<Connector port="8080" 
executor="tomcatThreadPool" 指定使用的线程池 
```
* 设置Tomcat的虚拟机运行参数
* linux的话在catalina.sh win 在.bat中调整JAVA_OPTS参数来让自己的机器发挥最好的性能。具体的参数规则如下（**十分重要，我的为数不多单机JVM调优的经验**）：

-server：表示这是应用于服务器的配置
（因为JVM有两种启动模式：client和server。server启动较慢，但性能好）

 -Xmx1024m：设置JVM最大可用内存为1024MB
 
 -Xms1024m：设置JVM最小内存为1024m。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。
 
 -Xmn1024m：设置JVM新生代大小（JDK1.4之后版本）。一般-Xmn的大小是-Xms的1/2左右
 
 -XX:NewSize：设置新生代大小
 
 -XX:MaxNewSize：设置最大的新生代大小
 
 -XX:PermSize：设置永久代大小(淘汰)
 
 -XX:MaxPermSize：设置最大永久代大小（淘汰）
 
 -XX:NewRatio=2：设置年轻代（包括 Eden 和两个 Survivor 区）与终身代的比值（除去永久代）。设置为 2，则年轻代与终身代所占比值为 1：2，年轻代占整个堆栈的 1/2（设置了这个老年代的大小也就确定了，一般是设置为2，1：1）
 
 -XX:MaxTenuringThreshold=10：设置垃圾最大年龄，默认为：15

## JVM调优经验
下面是我网找的一些配置参数的经验
修改 /usr/program/tomcat7/bin/catalina.sh 文件，把下面信息添加到文件第一行

如果服务器只运行一个 Tomcat

机子内存如果是 4G：
CATALINA_OPTS="-Dfile.encoding=UTF-8 -server -Xms2048m -Xmx2048m -Xmn1024m -XX:PermSize=256m -XX:MaxPermSize=512m -XX:SurvivorRatio=10 -XX:MaxTenuringThreshold=15 -XX:NewRatio=2 -XX:+DisableExplicitGC"

机子内存如果是 8G：
CATALINA_OPTS="-Dfile.encoding=UTF-8 -server -Xms4096m -Xmx4096m -Xmn2048m -XX:PermSize=256m -XX:MaxPermSize=512m -XX:SurvivorRatio=10 -XX:MaxTenuringThreshold=15 -XX:NewRatio=2 -XX:+DisableExplicitGC"

机子内存如果是 16G：
CATALINA_OPTS="-Dfile.encoding=UTF-8 -server -Xms8192m -Xmx8192m -Xmn4096m -XX:PermSize=256m -XX:MaxPermSize=512m -XX:SurvivorRatio=10 -XX:MaxTenuringThreshold=15 -XX:NewRatio=2 -XX:+DisableExplicitGC"

机子内存如果是 32G：
CATALINA_OPTS="-Dfile.encoding=UTF-8 -server -Xms16384m -Xmx16384m -Xmn8192m -XX:PermSize=256m -XX:MaxPermSize=512m -XX:SurvivorRatio=10 -XX:MaxTenuringThreshold=15 -XX:NewRatio=2 -XX:+DisableExplicitGC"

## 一个小坑哦
在现在已经不可按照上述方式来配置永久代大小了，在1.8之后，方法区不再存在于JAVA运行时数据区，而是被放到直接到内存当中啦（direct memory）

