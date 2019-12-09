# Java的三大集合

[toc]

## 总述
Java的三大集合分别指的是Map，List和Set。其中List和Set是实现了Collection接口。
Map接口的实现类主要有：HashMap，TreeMap，Hashtable，LinkedHashMap和ConcurrentHashMap。
Set接口的实现类主要有：HashSet，TreeSet和LinkedHashSet。
List接口的实现类主要有：ArrayList，LinkedList，Stack和Vector。

## 快速失败(fast-fail)机制
这是一种Java集合的错误检测机制。当多个线程对集合进行**结构**上的改变的操作时，**有可能**产生fail-fast。
**具体实现：**在迭代器遍历访问集合中的内容的时候，会使用一个modCount变量来检测遍历过程中内容是否发生了变化(结构改变)。如果是，modCount的值就会改变，迭代器在调用hasNext/next方法的时候，会判断modCount与expectedModCount是否一致，不一致的话(内容发生改变)就会抛出ConcurrModificationException。
这个机制能够在集合操作在真正遇到异常前就终止程序。

## Map
### HashMap和Hashtable的区别
- **线程安全：** HashMap是线程不安全的；Hashtable是线程安全的，Hashtable的方法使用了synchronized关键字来修饰，从而达到线程安全的效果。
- **对null key和null value的支持：** HashMap允许用null作为key，也允许用null作为value。Hashtable不允许，一旦put的key为null，就会抛出NullPointerException。
- **初始容量大小和每次扩容大小的不同：**Hastable的初始大小而11，每次扩容为2n+1；HashMap的初始大小为16，每次扩容为2n。
- **底层数据结构：**HashMap(JDK 1.8之前)和Hashtable的底层数据结构都是数组+链表。在JDK 1.8之后，链表长度大于8后，HashMap中的链表会转为红黑树。

#### 注意
Hashtable并不是绝对的线程安全。它的线程安全是指多线程的情况下，每个线程调用的Hashtable中相同的操作方法，如果有的线程调用了get方法，有的调用了remove方法，就会出现线程不安全的情况。
HashMap的线程不安全是在因为在**多线程环境下进行扩容**可能会出现**HashMap死循环**。

#### 为什么HashMap每次扩容都是2n
1. 因为容量是奇数的时候，长度的二进制表示会有0存在，浪费了空间。
2. 计算数组下标的时候，为了提高运算效率，会使用这个替代公式hash % length == hash & (length - 1)，而这个公式成立的前提是length是2的N次方。

length的值为2的N次幂有助于减少哈希碰撞的几率，同时提高空间利用率。
