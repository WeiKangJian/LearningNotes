# 深入锁和并发集合的核心——AQS（1）
@[TOC](目录)
## AQS的和ReentrantLock的关系
在Java提供的并发包下，会发现一大批并发集合，在这些集合中都用到了ReentrantLock的工具来保障线程安全。
(注意，lock的主体是AQS，而syncrynized的同步机制是靠监视器和底层指令实现的)。

先来看看在我们new一个ReentrantLock的对象，在调用lock()方法的时候究竟调用了什么。

当我们new一个ReentrantLock的对象的时候，要进行初始化首先有一个还未被初始化的核心成员变量sync(同步基元)。后面的lock等核心方法由它来实现。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191005184041365.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191005183445634.png)
再来看看ReentrantLock的构造器：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191005183810683.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)
其有两种构造器，在没有参数的时候默认new的的非公平的基元，在指定参数为true的时候则new的是公平的基元。那么这个sync究竟是怎么来的，其构造如何呢：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191005184236298.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191005184302169.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191005184320963.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)
如上面三张图所示，调用lock函数时的机制一目了然，在ReentrantLock中，先有了一个Sync静态内部类，其继承了AQS，再根据调用的是公平锁或者非公平锁，通过FairSync和NonfairSync来对lock()进行具体的实现。两个Sync的子类只覆盖了 **lock（）** 和  **tryAcquire（）** 这两个方法，即只在这两处有着不同的实现。

那么锁的核心实现就在AQS当中，下面来分析AQS。

## AQS的设计模式
**AQS**的实现是典型的**模板方法模式**将类申明为抽象类，即不能直接实例化AQS，需要实现其给定的几个模板方法。对这些方法的调用是在AQS中定义好了的。

eg: 在AQS之中上锁的很重要的一环是**acquire()**方法，该方法内部所需的是**tryAcquire（）**方法，而这个方法是需要在子类中定义的。所以如果实现AQS的子类没有重写 **tryAcquire（）** 的方法，将会抛出UnSupport的异常。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019100519111741.png)
## AQS主要字段（组成部分）
AQS的主要字段如下：

```
/**
 * 头节点指针，通过setHead进行修改
 */
private transient volatile Node head;

/**
 * 队列的尾指针
 */
private transient volatile Node tail;

/**
 * 同步器状态
 */
private volatile int state;
```

**其主要包含了一个FIFO的双向队列（CLH），一个同步器的状态state用来表示当前是否上锁。**
在Node的节点中主要包含字段如下：
SHARED	Node	new Node()	
一个标识，指示节点使用共享模式等待

EXCLUSIVE	Nodel	Null	
一个标识，指示节点使用独占模式等待

thread	Thread	Null	
当前Node操作的线程

nextWaiter	Node	Null	
指向下一个处于阻塞的节点


**此外，AQS本身也是继承了AOS，使其还有一个字段，即当前拥有锁的线程：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191005212024145.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191005212105764.png)
## AQS需要子类去实现的方法
1：tryAcquire
	以独占模式尝试获取锁，独占模式下调用acquire，尝试去设置state的值，如果设置成功则返回，如果设置失败则将当前线程加入到等待队列，直到其他线程唤醒。
	
2：tryRelease
	尝试独占模式下释放状态。

3：tryAcquireShared
	尝试在共享模式获得锁，共享模式下调用acquire，尝试去设置state的值，如果设置成功则返回，如果设置失败则将当前线程加入到等待队列，直到其他线程唤醒。

4：tryReleaseShared
	尝试共享模式下释放状态

5：isHeldExclusively
	是否是独占模式，表示是否被当前线程占用。

## AQS上锁原理
其上锁的过程是通过lock()的acquire的方法，去将state的状态通过CAS的形式进行改变，如果当前state是0，则独占该锁。如果不是0，则独占失败，进入CLH队列阻塞，等待被唤醒后占有锁。完成了上锁的过程。但其公平锁和非公平锁的实现方式由很大不同。

## 公平锁和非公平锁底层原理
### 公平锁
公平锁的实现很常见，就是先入先出，当前线程释放锁后，从头节点选取队列的第一个阻塞线程运行。其调用的方式如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191005210551701.png)
**直接进行acquire（）方法获取锁**，在进入acquire方法后：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191005210646267.png)
进行tryAcquire的方法，**（这个方法是被FairSyn已经重写过的了（模板实现））**，

如果获取成功，将当前线程设置为锁拥有者线程。

如果获取失败，即已经被上锁了（tryAcquire返回flase），就执行addWaiter，即将线程阻塞，设置为独占（排他），加入CLH队尾（addWaiter实现）。

公平锁的tryAcquire（）方法如下，注意深色那一行，和非公平锁的方式显著不同。该处要判断如果CLH队列中有等待的线程（不为空），那么直接获取锁失败，进入队尾部乖乖等待。这个是公平锁的底层方式。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191005211215691.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)
### 非公平锁
同样是上面的公平锁的实现方式进行对比，首先在lock层面即有不同，非公平锁，在加锁的时候即进行一次抢占，如果此时CLH中运行的线程刚好释放锁，那么此时，该线程就能获得锁，实现新线程的抢占。如果此次抢占不成功，进行acquire的方法。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019100521330495.png)
这里的acquire的方法和公平锁的方式一致，但方法内部的tryAcquire却有着不同的实现方式。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191005213639537.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191005213808108.png)

对比公平锁和非公平锁的tryAcquire()方式（下图），发现此处非公平锁又多了一次抢占机会（上一次是在lock中），当判断c==0的时候，此时还锁是未占有的状态，则直接尝试占有锁，不考虑等待队列空不空（公平锁是等待队列不为空就放弃占有）。此时新线程能抢占锁。

**注意：当此处判断state即c不为0的时候，同样占有失败，！tryAcquire（）返回true，执行addwaiter方法，将此线程加入CLH的队列尾部。**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191005213615479.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)

### 总结
即非公平锁的不公平体现在新线程在获取锁的时候，如果锁这时候刚好被释放，直接抢占锁，是针对队列中其他等待线程的不公平。如果新线程获取锁失败，也是要加入CLH的队尾部的，对于其等待队列中的所有线程之间来说，仍然是公平的。
