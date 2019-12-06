*2019.9.26**
   * [LinkHashMap]()
   * [TreeMap&amp;&amp;TreeSet]()
      * [Treemap和TreeSet实现原理]()
      * [关于红黑树的性质]()
      * [为什么使用红黑树而不是AVL（平衡二叉查找树）]()
      * [举个栗子（TreeMap的put）]()
# LinkHashMap
其实LinkHashMap和HashMap的差别并没有很大，
其实Entry甚至是继承的HashMap，但是在原有的Entry中新增加了两个引用，*befor*和*After*，
一个指向前一个插入的entry一个指向后一个插入的entry，构成了一个双向链表。轻松实现了
HashMap按照插入的顺序进行排列。

# TreeMap&&TreeSet
## Treemap和TreeSet实现原理
**TreeMap和TreeSet实现机制是一样的都是采用红黑树，RB树天生就具备排序的特性（中序遍历）**
TreeMap是实现自SortMap的接口的，和HashMap的数据结构具有天壤之别。其是采用红黑树来进行实现的。红黑树是一种平衡的二叉查找树。在TreeMap的实现过程中，是默认了在没有外部比较器的情况下，通过对key值从小到大进行排序，其时间复杂度时logN
所以TreeMap的迭代，查找，都是基于红黑树来进行实现的，树里面的每个节点都是一个Entry.

## 关于红黑树的性质

红黑树的特性:
（1）每个节点或者是黑色，或者是红色。
（2）根节点是黑色。
（3）每个叶子节点（NIL）是黑色。 [注意：这里叶子节点，是指为空(NIL或NULL)的叶子节点！]
（4）如果一个节点是红色的，则它的子节点必须是黑色的。
（5）从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。[这里指到叶子节点的路径]

## 为什么使用红黑树而不是AVL（平衡二叉查找树）

先看两种树有哪些不同：
首先AVL树是强平衡的，即要求每个节点的左右子树的差小于等于1.
但是：红黑树不追求"完全平衡"，即不像AVL那样要求节点的 |balFact| <= 1，它只要求部分达到平衡，但是提出了为节点增加颜色，红黑是用非严格的平衡来换取增删节点时候旋转次数的降低，任何不平衡都会在三次旋转之内解决，而AVL是严格平衡树，因此在增加或者删除节点的时候，根据不同情况，旋转的次数比红黑树要多。
![在这里插入图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTY3NzkxNC8yMDE5MDcvMTY3NzkxNC0yMDE5MDcyMTE2MjY0ODEzMS0zMjY5OTYwMzAucG5n?x-oss-process=image/format,png)
所以在使用的过程中，若搜索的次数远远大于插入和删除，那么选择AVL，如果搜索，插入删除次数几乎差不多，应该选择红黑树。至于不同的二叉查找树，可能退化位链表，不用。
在TREEMAP中，考虑到插入PUT和删除节点的次数比较多，就使用的红黑树。

## 举个栗子（TreeMap的put）
![在这里插入图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTY3NzkxNC8yMDE5MDcvMTY3NzkxNC0yMDE5MDcyMTE2MjcwMDc0Ni0xNTQyNDY3MzU0LnBuZw?x-oss-process=image/format,png)
