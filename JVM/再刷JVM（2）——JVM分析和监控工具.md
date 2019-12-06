 **2019.11.14**
   * [常见的查看当前机器java参数的命令](#常见的查看当前机器java参数的命令)
   * [JDK 监控和故障处理工具总结](#jdk-监控和故障处理工具总结)
      * [JDK 命令行工具](#jdk-命令行工具)
         * [jps:查看所有 Java 进程](#jps查看所有-java-进程)
         * [jstat: 监视虚拟机各种运行状态信息](#jstat-监视虚拟机各种运行状态信息)
         * [ jinfo: 实时地查看和调整虚拟机各项参数](#-jinfo-实时地查看和调整虚拟机各项参数)
      * [JDK 可视化分析工具](#jdk-可视化分析工具)
         * [JConsole:Java 监视与管理控制台](#jconsolejava-监视与管理控制台)
            * [连接 Jconsole](#连接-jconsole)
            * [查看 Java 程序概况](#查看-java-程序概况)
            * [内存监控](#内存监控)
            * [线程监控](#线程监控)
         * [Visual VM:多合一故障处理工具](#visual-vm多合一故障处理工具

在我们运行我们的Java程序时，我们需要实时关注系统的运行情况，来发现问题，和优化系统，并能分析处死锁，空间不足等问题，所以学会分析故障，尤其是在linux环境下监控jvm的运行是十分有必要的，下面是我在参考了网上一些文档和博客后，经过实践总结的一些方法：
***
# 常见的查看当前机器java参数的命令
 **java -XX:+PrintCommandLineFlags -version**
查看虚拟机相关信息，包括使用的垃圾回收器，当前最小堆（初始），最大堆的大小

**java-XX:MaxTenuringThreshold** 
更改对象GC年龄的阈值，默认是15

# JDK 监控和故障处理工具总结

## JDK 命令行工具

这些命令在 JDK 安装目录下的 bin 目录下，是安装好java 环境后就带有的天然命令：

- **`jps`** (JVM Process Status）: 类似 UNIX 的 `ps` 命令。用户查看所有 Java 进程的启动类、传入参数和 Java 虚拟机参数等信息；
- **`jstat`**（ JVM Statistics Monitoring Tool）:  用于收集 HotSpot 虚拟机各方面的运行数据;
- **`jinfo`** (Configuration Info for Java) : Configuration Info forJava,显示虚拟机配置信息;


### `jps`:查看所有 Java 进程

`jps`(JVM Process Status) 命令类似 UNIX 的 `ps` 命令。

`jps`：显示虚拟机执行主类名称以及这些进程的本地虚拟机唯一 ID（Local Virtual Machine Identifier,LVMID）。`jps -q` ：只输出进程的本地虚拟机唯一 ID。

```powershell
C:\Users\SnailClimb>jps
7360 NettyClient2
17396
7972 Launcher
16504 Jps
17340 NettyServer
```

`jps -l`:输出主类的全名，如果进程执行的是 Jar 包，输出 Jar 路径。

```powershell
C:\Users\SnailClimb>jps -l
7360 firstNettyDemo.NettyClient2
17396
7972 org.jetbrains.jps.cmdline.Launcher
16492 sun.tools.jps.Jps
17340 firstNettyDemo.NettyServer
```

`jps -v`：输出虚拟机进程启动时 JVM 参数。

`jps -m`：输出传递给 Java 进程 main() 函数的参数。

### `jstat`: 监视虚拟机各种运行状态信息

jstat（JVM Statistics Monitoring Tool） 使用于监视虚拟机各种运行状态信息的命令行工具。 它可以显示本地或者远程（需要远程主机提供 RMI 支持）虚拟机进程中的类信息、内存、垃圾收集、JIT 编译等运行数据，在没有 GUI，只提供了纯文本控制台环境的服务器上，它将是运行期间定位虚拟机性能问题的首选工具。

**`jstat` 命令使用格式：**

```powershell
jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
```

比如 `jstat -gc -h3 31736 1000 10`表示分析进程 id 为 31736 的 gc 情况，每隔 1000ms 打印一次记录，打印 10 次停止，每 3 行后打印指标头部。

**常见的 option 如下：**

- `jstat -class vmid` ：显示 ClassLoader 的相关信息；
- `jstat -compiler vmid` ：显示 JIT 编译的相关信息；
- `jstat -gc vmid` ：显示与 GC 相关的堆信息；
- `jstat -gccapacity vmid` ：显示各个代的容量及使用情况；
- `jstat -gcnew vmid` ：显示新生代信息；
- `jstat -gcnewcapcacity vmid` ：显示新生代大小与使用情况；
- `jstat -gcold vmid` ：显示老年代和永久代的信息；
- `jstat -gcoldcapacity vmid` ：显示老年代的大小；
- `jstat -gcpermcapacity vmid` ：显示永久代大小；
- `jstat -gcutil vmid` ：显示垃圾收集信息；

另外，加上 `-t`参数可以在输出信息上加一个 Timestamp 列，显示程序的运行时间。

### ` jinfo`: 实时地查看和调整虚拟机各项参数

`jinfo vmid` :输出当前 jvm 进程的全部参数和系统属性 (第一部分是系统的属性，第二部分是 JVM 的参数)。

`jinfo -flag name vmid` :输出对应名称的参数的具体值。比如输出 MaxHeapSize、查看当前 jvm 进程是否开启打印 GC 日志 ( `-XX:PrintGCDetails` :详细 GC 日志模式，这两个都是默认关闭的)。

```powershell
C:\Users\SnailClimb>jinfo  -flag MaxHeapSize 17340
-XX:MaxHeapSize=2124414976
C:\Users\SnailClimb>jinfo  -flag PrintGC 17340
-XX:-PrintGC
```

使用 jinfo 可以在不重启虚拟机的情况下，可以动态的修改 jvm 的参数。尤其在线上的环境特别有用,请看下面的例子：

`jinfo -flag [+|-]name vmid` 开启或者关闭对应名称的参数。

```powershell
C:\Users\SnailClimb>jinfo  -flag  PrintGC 17340
-XX:-PrintGC

C:\Users\SnailClimb>jinfo  -flag  +PrintGC 17340

C:\Users\SnailClimb>jinfo  -flag  PrintGC 17340
-XX:+PrintGC
```


## JDK 可视化分析工具

### JConsole:Java 监视与管理控制台

直接在命令行输入：Jconsole

#### 连接 Jconsole

![连接 Jconsole](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9teS1ibG9nLXRvLXVzZS5vc3MtY24tYmVpamluZy5hbGl5dW5jcy5jb20vMjAxOS02LzFKQ29uc29sZSVFOCVCRiU5RSVFNiU4RSVBNS5wbmc?x-oss-process=image/format,png)

如果需要使用 JConsole 连接远程进程，可以在远程 Java 程序启动时加上下面这些参数:

```properties
-Djava.rmi.server.hostname=外网访问 ip 地址 
-Dcom.sun.management.jmxremote.port=60001   //监控的端口号
-Dcom.sun.management.jmxremote.authenticate=false   //关闭认证
-Dcom.sun.management.jmxremote.ssl=false
```

在使用 JConsole 连接时，远程进程地址如下：

```
外网访问 ip 地址:60001 
```

#### 查看 Java 程序概况

![查看 Java 程序概况 ](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9teS1ibG9nLXRvLXVzZS5vc3MtY24tYmVpamluZy5hbGl5dW5jcy5jb20vMjAxOS02LzIlRTYlOUYlQTUlRTclOUMlOEJKYXZhJUU3JUE4JThCJUU1JUJBJThGJUU2JUE2JTgyJUU1JTg2JUI1LnBuZw?x-oss-process=image/format,png)

#### 内存监控

JConsole 可以显示当前内存的详细信息。不仅包括堆内存/非堆内存的整体信息，还可以细化到 eden 区、survivor 区等的使用情况，如下图所示。

点击右边的“执行 GC(G)”按钮可以强制应用程序执行一个 Full GC。

> - **新生代 GC（Minor GC）**:指发生新生代的的垃圾收集动作，Minor GC 非常频繁，回收速度一般也比较快。
> - **老年代 GC（Major GC/Full GC）**:指发生在老年代的 GC，出现了 Major GC 经常会伴随至少一次的 Minor GC（并非绝对），Major GC 的速度一般会比 Minor GC 的慢 10 倍以上。

![内存监控 ](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9teS1ibG9nLXRvLXVzZS5vc3MtY24tYmVpamluZy5hbGl5dW5jcy5jb20vMjAxOS02LzMlRTUlODYlODUlRTUlQUQlOTglRTclOUIlOTElRTYlOEUlQTcucG5n?x-oss-process=image/format,png)

#### 线程监控

类似我们前面讲的 `jstack` 命令，不过这个是可视化的。

最下面有一个"检测死锁 (D)"按钮，点击这个按钮可以自动为你找到发生死锁的线程以及它们的详细信息 。

![线程监控 ](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9teS1ibG9nLXRvLXVzZS5vc3MtY24tYmVpamluZy5hbGl5dW5jcy5jb20vMjAxOS02LzQlRTclQkElQkYlRTclQTglOEIlRTclOUIlOTElRTYlOEUlQTcucG5n?x-oss-process=image/format,png)

### Visual VM:多合一故障处理工具

VisualVM 提供在 Java 虚拟机 (Java Virutal Machine, JVM) 上运行的 Java 应用程序的详细信息。在 VisualVM 的图形用户界面中，您可以方便、快捷地查看多个 Java 应用程序的相关信息。Visual VM 官网：<https://visualvm.github.io/> 。Visual VM 中文文档:<https://visualvm.github.io/documentation.html>。

下面这段话摘自《深入理解 Java 虚拟机》。

> VisualVM（All-in-One Java Troubleshooting Tool）是到目前为止随 JDK 发布的功能最强大的运行监视和故障处理程序，官方在 VisualVM 的软件说明中写上了“All-in-One”的描述字样，预示着他除了运行监视、故障处理外，还提供了很多其他方面的功能，如性能分析（Profiling）。VisualVM 的性能分析功能甚至比起 JProfiler、YourKit 等专业且收费的 Profiling 工具都不会逊色多少，而且 VisualVM 还有一个很大的优点：不需要被监视的程序基于特殊 Agent 运行，因此他对应用程序的实际性能的影响很小，使得他可以直接应用在生产环境中。这个优点是 JProfiler、YourKit 等工具无法与之媲美的。

 VisualVM 基于 NetBeans 平台开发，因此他一开始就具备了插件扩展功能的特性，通过插件扩展支持，VisualVM 可以做到：

- **显示虚拟机进程以及进程的配置、环境信息（jps、jinfo）。**
- **监视应用程序的 CPU、GC、堆、方法区以及线程的信息（jstat、jstack）。**
- **dump 以及分析堆转储快照（jmap、jhat）。**
- **方法级的程序运行性能分析，找到被调用最多、运行时间最长的方法。**
- **离线程序快照：收集程序的运行时配置、线程 dump、内存 dump 等信息建立一个快照，可以将快照发送开发者处进行 Bug 反馈。**
- **其他 plugins 的无限的可能性......**



