
   * [HashMap底层原理初探]()
      * [1：HashMap的数据结构]()
      * [2：HashMap的散列算法]()
      * [3：HashMap的扩容机制]()
      * [4：HashMap的get和put]()
         * [put]()
         * [get]()
      * [5：HashMap和HashTale和ConcurrentHashmap的区别联系]()

# HashMap底层原理初探
**2019.9.25**
## 1：HashMap的数据结构
在实现HashMap底层的时候，在1.7之前是数组加链表的形式。对每一个Key的对象，先进行一次Hash,对于得到的Hash(整形值)，散列到桶数组上。在其中存放的是node的节点，其头节点放在数组里，后面是冲突后加进来的其他链表。在1.8之后，这个链表的长度大于8会转换成红黑树，加快速率。

为什么使用红黑树不适用其他二叉查找树：二叉查找树再最坏的情况下会退化成位一个单链表，加大的查找的难度。
![数据结构示意](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jYW1vLmdpdGh1YnVzZXJjb250ZW50LmNvbS83MDYyMmE2MmVlMjYyZmI2MDg5NmY1ZTYyOTAxMmVhYmQ2OThjYjYwLzY4NzQ3NDcwNzMzYTJmMmY2ZDc5MmQ2MjZjNmY2NzJkNzQ2ZjJkNzU3MzY1MmU2ZjczNzMyZDYzNmUyZDYyNjU2OTZhNjk2ZTY3MmU2MTZjNjk3OTc1NmU2MzczMmU2MzZmNmQyZjMyMzAzMTM5MmQzNzJmNmE2NDZiMzEyZTM4MjU0NTM0MjU0MjM5MjUzODQyMjU0NTM1MjUzODM5MjUzODQ0MjU0NTM3MjUzOTQxMjUzODM0MjU0NTM1MjUzODM2MjUzODM1MjU0NTM5MjUzODMzMjU0MTM4MjU0NTM3MjU0MjQyMjUzOTMzMjU0NTM2MjUzOTQ1MjUzODM0MmU3MDZlNjc?x-oss-process=image/format,png)
![在这里插入图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jYW1vLmdpdGh1YnVzZXJjb250ZW50LmNvbS8yMGRlN2U0NjVjYWMyNzk4NDI4NTEyNThlYzRkMWVjMWM0ZDNkN2QxLzY4NzQ3NDcwM2EyZjJmNmQ3OTJkNjI2YzZmNjcyZDc0NmYyZDc1NzM2NTJlNmY3MzczMmQ2MzZlMmQ2MjY1Njk2YTY5NmU2NzJlNjE2YzY5Nzk3NTZlNjM3MzJlNjM2ZjZkMmYzMTM4MmQzODJkMzIzMjJmMzYzNzMyMzMzMzM3MzYzNDJlNmE3MDY3?x-oss-process=image/format,png)
## 2：HashMap的散列算法
散列算法：在JDK1.7之前散列的算法是简单的对桶数组的长度（Node.length）取模，这样能分布的更均匀。但取模的开销较大。在1.7之后，采取了全新的散列算法：

JDK1.7之后：通过高位运算和&数组长度的方式实现。具体来说是将得到的Hash的值向有移32位。和原来的Hash进行一次异或运算后再和（n-1)进行&运算。即K=（n-1)&(h^h>>>16)

## 3：HashMap的扩容机制
在算法层面：
NODE数组的长度默认是16(length)
在HASHMAP中还定义了以下3各参数
length node数组的长度
负载因子
最大存入的键值对个数
length*负载因子等于最大存入个数
当存取的键值对超过这个个数后。node数组会扩充为原来的两倍。这就是为什么
桶数组是2的整数次方的原因。进行扩容，会伴随着一次重新hash分配，并且会遍历hash表中所有的元素，是非常耗时的。在编写程序中，要尽量避免resize。

## 4：HashMap的get和put
### put
HashMap只提供了put用于添加元素，putVal方法只是给put方法调用的一个方法，并没有提供给用户使用。

对putVal方法添加元素的分析如下：

①如果定位到的数组位置没有元素 就直接插入。

②如果定位到的数组位置有元素就和要插入的key比较，如果key相同就直接覆盖，如果key不相同，就判断p是否是一个树节点，如果是就调用e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value)将元素添加进入。如果不是就遍历链表插入(插入的是链表尾部)。

![在这里插入图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jYW1vLmdpdGh1YnVzZXJjb250ZW50LmNvbS9kZjFjMzA3N2I5Mjk4NzM3MjdjOTk3MGUzYTQ4YzBlZjE0ZmQwOTRkLzY4NzQ3NDcwNzMzYTJmMmY2ZDc5MmQ2MjZjNmY2NzJkNzQ2ZjJkNzU3MzY1MmU2ZjczNzMyZDYzNmUyZDYyNjU2OTZhNjk2ZTY3MmU2MTZjNjk3OTc1NmU2MzczMmU2MzZmNmQyZjMyMzAzMTM5MmQzNzJmNzA3NTc0MjU0NTM2MjUzOTM2MjU0MjM5MjU0NTM2MjU0MjMzMjUzOTM1MmU3MDZlNjc?x-oss-process=image/format,png)
### get

```
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 数组元素相等
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 桶中不止一个节点
        if ((e = first.next) != null) {
            // 在树中get
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 在链表中get
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```
## 5：震惊！多线程下使用HashMap会导致死循环而让CPU 100%
  在JDK7中，每次当MAP的容量超过最大容量后，会进行一次扩容。（这里注意，扩容是加完新元素后进行的扩容）在执行rsize的过程中，会遍历原有的MAP，重新进行一次ReHash的过程，这里的算法是遍历一个节点即加进对应的新的MAP的桶数组中，如果已经有了，类似于头插法，即和原链表是逆着的。如果同时有多个线程进行ReHash的过程，可能会带来死循环。
  
 假设有两个线程
 ```
 do {
    Entry<K,V> next = e.next; //  假设线程一执行到这里就被调度挂起了，而此时线程二已经完成了一次rehash
    int i = indexFor(e.hash, newCapacity);
    e.next = newTable[i];
    newTable[i] = e;
    e = next;
} while (e != null);
```

在之后参考（https://www.cnblogs.com/williamjie/p/11089522.html）的图片演示，会导致出现环形链表。

在JDK1.8之后，通过限制顺序，修复了这个问题，但在多线程下PUT的时候，仍会数据丢失，这时候就要使用线程安全的同步类和并发类啦

## 6：HashMap和HashTale和ConcurrentHashmap的区别联系
[后边会有一篇文章专门介绍](https://mp.csdn.net).

