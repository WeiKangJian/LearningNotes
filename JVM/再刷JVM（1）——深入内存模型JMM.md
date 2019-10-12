# 再刷JVM（1）——深入内存模型JMM
## 运行时数据区一览
![在这里插入图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jYW1vLmdpdGh1YnVzZXJjb250ZW50LmNvbS8wYmNjNmMwMWE5MTliMTc1ODI3ZjBkNTU0MGFlZWMxMTVkZjZjMDAxLzY4NzQ3NDcwNzMzYTJmMmY2ZDc5MmQ2MjZjNmY2NzJkNzQ2ZjJkNzU3MzY1MmU2ZjczNzMyZDYzNmUyZDYyNjU2OTZhNjk2ZTY3MmU2MTZjNjk3OTc1NmU2MzczMmU2MzZmNmQyZjMyMzAzMTM5MmQzMzRhNjE3NjYxZThiZjkwZThhMThjZTY5N2I2ZTY5NWIwZTY4ZGFlZTU4Y2JhZTU5ZjlmNGE0NDRiMzEyZTM4MmU3MDZlNjc?x-oss-process=image/format,png)
已经看过一遍深入虚拟机，上面的运行时数据区域不会陌生。
值得注意的一点是：在1.8之前，方法区（永生代）是存储在运行时数据区的，而在1.8之后，方法区被迁移到了直接内存，即在JAVA进程之外的主机内存里，这里是不会发生GC的。

### **线程独有**
**程序计数器**（在切换时保存当前线程执行到哪一步，指向下一条被执行的指令。通过改变程序计数器来实现循环，分支，跳转等，不会发生OOM）

**虚拟机栈**（保存局部变量的地方（局部变量表）（孤尽老师说每个局部变量表是一个抽屉，可以单独在抽屉中运算如i++），操作栈，和方法返回地址（返回或者异常上层处理））（局部变量是一个整体的概念，如一个局部变量是 Student s =new Student(),局部变量表中存的是**s:s的地址**，统称为局部变量）（OOM和STACKOVERFLOW）

**本地方法栈**（native方法）

### **线程共享**
**堆**：new的对象所处的地方，不是所有对象都在堆中（如1.7以前的超类Class，由字节码生成Class对象后存在方法区）

**方法区**：在1.8之前是在运行时数据区，1.8之后再直接内存（元空间）存放类元数据，xclass，包括类的静态属性，方法，构造器等信息。

**常量池**：1.8之前还在方法区，1.8之后在堆中单独开放的区域。
![在这里插入图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jYW1vLmdpdGh1YnVzZXJjb250ZW50LmNvbS8xNzYyMDcyMWE5ZjMyNmEyMzVhZWVjODk1Njk0OWNlYzAzZjNmMTI1LzY4NzQ3NDcwM2EyZjJmNmQ3OTJkNjI2YzZmNjcyZDc0NmYyZDc1NzM2NTJlNmY3MzczMmQ2MzZlMmQ2MjY1Njk2YTY5NmU2NzJlNjE2YzY5Nzk3NTZlNjM3MzJlNjM2ZjZkMmYzMTM4MmQzOTJkMzEzNDJmMzIzNjMwMzMzODM0MzMzMzJlNmE3MDY3?x-oss-process=image/format,png)

### 堆中细致的分代机制和垃圾回收算法
进一步划分的目的是更好地回收内存，或者更快地分配内存。堆中划分如下：
![在这里插入图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jYW1vLmdpdGh1YnVzZXJjb250ZW50LmNvbS80MDEyNDgyZjQ5OTI2YjM1ZDg1NTdiNjM5NTJlZTYwNWZkMjU5ZjYyLzY4NzQ3NDcwNzMzYTJmMmY2ZDc5MmQ2MjZjNmY2NzJkNzQ2ZjJkNzU3MzY1MmU2ZjczNzMyZDYzNmUyZDYyNjU2OTZhNjk2ZTY3MmU2MTZjNjk3OTc1NmU2MzczMmU2MzZmNmQyZjMyMzAzMTM5MmQzM2U1YTA4NmU3YmI5M2U2OWU4NDJlNzA2ZTY3?x-oss-process=image/format,png)
新生的对象一般在eden中，也有大对象可直接进入老年代的。新生对象在年轻代中，经过一次YGC存活后进入s0，然后如果还能存活，在s0,s1之间来回交换，（即典型的复制算法，将存活的对象全复制过去，然后清空原来的s0或者s1。达到一定次数15。（每个对象都有一个计数器，藏在对象头中）转移进老年代中，老年代中的对象轻易不会被清理，除非发生FULLGC）

除了上面提到的复制算法，还有标记——清除（产生碎片空间），标记——整理（开销较大）（常用在FGC中）。
对象GC的流程图机制如下图（摘自码出高效）
注意新对象如果eden放不下，1次YGC，再判断，还放不下，进入老年代，放不下一次FGC，再放不下，OOM。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191012215038350.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)
### 判断是否回收对象的方式
 （1）引用计数法，对被引用的对象，计数+1，可能导致互相引用，内存泄漏
 （2）引用可达法：GCroot点，虚拟机栈和本地方法栈中的变量和常量池中的引用，如果引用可达，不回收


##  深入new对象的过程
 从老生常谈的生成对象的5个步骤说起：
 ### 类加载检查&&类加载器&&双亲委派模型
 **类加载检查：**
虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载过、解析和初始化过。如果没有，那必须先执行相应的类加载过程。
类加载的过程包括三个过程：**加载，链接，初始化**
可以描述为将字节码（即指令，16进制，1个指令对应两个字节）实例化为Class对象，其中包括类的属性，方法等信息。在链接阶段进行验证，准备，解析，初始化阶段执行类构造器的Cinit方法。Class对象和反射的关系紧密，是反射的基础，甚至可以通过其newInstance来创造对象，（这种形式只能创建无参对象）。这时候如果需要加载父类会先加载父类。**在初始化的时候，会执行静态代码块**。
**类加载器**
**双亲委派模型**
 ### 分配内存
 ### 初始化零值
 ### 生成对象头
 ### 执行init方法

