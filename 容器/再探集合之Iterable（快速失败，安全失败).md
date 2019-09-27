# 再探集合组件之Iterable
Iterator是定义在java.util下面的一个接口，定义了需要实现的几个方法。
**hasNext**      
**next**
**remove**

在各种集合中通过内部类实现了这个接口，并实现了里面的方法

在Java.util下面的所有集合调用这个Iterator方法获取取迭代器进行操作的时候，
通过next().hasnext()方法来进行操作。这里的操作都是在原来集合上的操作。在这个Iterator的接口上除了next等操作。只定义了remove。

因为Java.util是的迭代器是在原集合上操作的，在Util下所有的低级集合中实现Iterator接口是用一个内部类来实现的接口，在其实现的过程中都定义了一个叫做**expectedModCount**的东西：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190927210215312.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)
 
它一开始被赋予的是该内部类之外的modCount的变量，当它每次进行next和Remove之前都要进行一次checkForComodification()的检查。一旦modCount和expectModCount不等，便会报出异常。注意：使用Iterator本身的remove方法在单线程环境中是正确的，因为它在进行一次这样的操作（也是通过调用this，然后让原集合删除），但在之后又重新对expectCount赋予modCount的值。使其不会报错。单在迭代的过程中，调用原集合的remove方法，只让modCount++；就会使得两个值不等，报出ConcurrentModificationException异常。所以在迭代的过程中，只能用迭代器的remove方法，且不可进行原集合上的add()等操作.
 

## 快速失败和安全失败
在多线程环境中，使用这一类低级集合，哪怕使用Iterator自带的remove也很快会导致expectCount和modCount值不相同的情况，抛出异常，称之为快速失败。
  而在高级集合，比如CopyOnWriteList和CopyOnWriteSet之中，其实现的方式已经不是继承自AbstractList了，而是仅仅实现了List的接口，就不再有modCount这个玩意。而且Iterator对其的操作是在其拷贝上进行的，即不会抛出这个异常。称之为安全失败。

## Iterator和ListIterator关系
几乎所有的集合类（包括同步和并发集合）都以内部类的形式实现了这两个接口。观察ListIterator接口，可以发现其比Iterator多了好几个方法，比如add,set,haspervious .pervious等，可以向前迭代。可以添加，其他和Iteritor一样，同样会造成快速失败和安全失败。当然，其本身的add（）方法在单线程环境下也不会报出异常。
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190927210236929.jpg)

