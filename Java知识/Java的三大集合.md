# Java的三大集合

[toc]

# 总述
Java的三大集合分别指的是Map，List和Set。其中List和Set是实现了Collection接口。
Map接口的实现类主要有：HashMap，TreeMap，Hashtable，LinkedHashMap和ConcurrentHashMap。
Set接口的实现类主要有：HashSet，TreeSet和LinkedHashSet。
List接口的实现类主要有：ArrayList，LinkedList，Stack和Vector。

# 快速失败(fast-fail)机制
这是一种Java集合的错误检测机制。当多个线程对集合进行**结构**上的改变的操作时，**有可能**产生fail-fast。
**具体实现：**在迭代器遍历访问集合中的内容的时候，会使用一个modCount变量来检测遍历过程中内容是否发生了变化(结构改变)。如果是，modCount的值就会改变，迭代器在调用hasNext/next方法的时候，会判断modCount与expectedModCount是否一致，不一致的话(内容发生改变)就会抛出ConcurrModificationException。
这个机制能够在集合操作在真正遇到异常前就终止程序。

# Map
## HashMap和Hashtable的区别
- **线程安全：** HashMap是线程不安全的；Hashtable是线程安全的，Hashtable的方法使用了synchronized关键字来修饰，从而达到线程安全的效果。
- **对null key和null value的支持：** HashMap允许用null作为key，也允许用null作为value。Hashtable不允许，一旦put的key为null，就会抛出NullPointerException。
- **初始容量大小和每次扩容大小的不同：**Hastable的初始大小而11，每次扩容为2n+1；HashMap的初始大小为16，每次扩容为2n。
- **底层数据结构：**HashMap(JDK 1.8之前)和Hashtable的底层数据结构都是数组+链表。在JDK 1.8之后，链表长度大于8后，HashMap中的链表会转为红黑树。

### 注意
Hashtable并不是绝对的线程安全。它的线程安全是指多线程的情况下，每个线程调用的Hashtable中相同的操作方法，如果有的线程调用了get方法，有的调用了remove方法，就会出现线程不安全的情况。
HashMap的线程不安全是在因为在**多线程环境下进行扩容**可能会出现**HashMap死循环**。

### 为什么HashMap每次扩容都是2n
1. 因为容量是奇数的时候，长度的二进制表示会有0存在，浪费了空间。
2. 计算数组下标的时候，为了提高运算效率，会使用这个替代公式hash % length == hash & (length - 1)，而这个公式成立的前提是length是2的N次方。

length的值为2的N次幂有助于减少哈希碰撞的几率，同时提高空间利用率。

### 解决Hash冲突的方法有哪些
- 拉链法(HashMap使用的方法)
- 线性探测再散列法：第一次Hash计算得到的下标为i，表长为n，Hash(( i + 1 ) % n)。
- 二次探测再散列法：序列：12，-12，22，-22...2k，-2k(k <= n/2)，Hash((i +- 2k) % n)。
- 伪随机探测再散列法：构造一个伪随机数序列：(2，5，9...，p)，Hash((i + p) % n)。

## ConcurrentHashMap
相比起Hashtable的全局锁，该集合类使用的使分段锁。在操作该类的对象的时候只会锁住操作的部分。

## TreeMap
底层数据结构是红黑树，该集合的键值是按键来排序的。如果放入的键是自定义引用类型，那么该类要实现Comparable接口，并实现compareTo方法。

### Comparable和Comparator的区别
- 两者都是接口
- Comparable定义的是对象的比较规则，在使用Collections.sort或Arrays.sort等方法的时候，直接传入对应的存放有自定义对象的集合就可以了。
- 以上两个方法也支持开发人员传入自己定制的Comparator，按照自己给自定义对象排序。

# List
## ArrayList和LinkedList的区别
- **线程安全：**两者都是非线程安全的。
- **数据结构：**前者底层是Object数组，后者是双向链表。
- **插入和删除操作：**前者的插入和删除的时间复杂度受元素所在位置的影响；后者不受影响。
- **快速随机访问：**前者实现了RandomAccess接口(一个空接口)，能够高效的随机访问元素；后者不行。
- **内存占用：**前者会在List的结尾预留一定的容量空间；后者每个元素会消耗较多的空间。

# Set
## HashSet和TreeSet的区别
HashSet的底层是Hash表，TreeSet的底层是红黑树。
- HashSet实际上是HashMap，HashSet的put操作就是key = e，value = PRESENT。
- HashSet的contains就是使用了HashMap的containsKey方法。

# LinkedHashMap和LinkedHashSet
LinkedHashMap的Entry元素中比HashMap多了before和after这两个属性，这样就能够构建一个双向链表从而保持元素的插入顺序。
使用public LinkedHashMap(int initialCapacity,float loadFactor,boolean accessOrder)，**accessOrder传入true可以实现LRU缓存算法(顺序访问)**。
与HashMap和HashSet类似，LinkedHashSet的底层使用LinkedHashMap实现的。

# List和Set的区别
- List中的元素是有序且允许重复的。
- Set中的元素是无序的且不能够重复。

# Iterator和ListIterator的区别
- 前者能够用来遍历List和Set集合，后者只能用来遍历List集合。
- 前者只能前向遍历，后者可以前向遍历和后向遍历。
- 后者是前者的实现，并增加了新的功能。

# 数组和集合之间的转换
- 数组 -> 集合：List<String> asList = Arrays.asList(arr2); 得到的对象是一个Arrays内部私有的ArrayList和一般的ArrayList类不同，不支持修改操作。
- 集合 -> 数组：Object[] arr = list.toArray();

