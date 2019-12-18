# Android必知必会
[toc]

## Dalvik和ART的区别
### Dalvik虚拟机
该虚拟机时Google公司设计的用于Android平台的Java虚拟机。它支持dex格式的Java应用程序运行，dex格式是专为Dalvik应用设计的一种**压缩格式**，适合**内存和处理器速度有限的系统**。Dalvik经过优化，允许在有限的内存中**同时运行多个虚拟机的实例**，并且每一个Dalvik应用作为独立的**Linux进程执行**。独立的进程可以**防止**在虚拟机崩溃的时候所有程序被关闭。
### ART(AndroidRunTime)
Dalvik是依靠一个**JIT(Just-In-Time)编译器**去解释字节码，运行时编译后的应用代码都需要通过一个解释器在用户的设备上运行，这一机制不高效，但是**能让应用更容易在不同硬件和架构上运行**。
而ART则在应用安装的时候就**预编译字节码到机器语言**，该机制叫**AOT(Ahead-Of-Time)预编译**。这样应用程序执行将**更有效率，启动更快**。
### 两者的区别
- Dalvik的安装速度快，但是应用的运行效率低；ART则是安装速度和第一次启动都会比较慢，但是以后每次可以直接执行，效率高。
- ART的占用空间比Dalvik大**(因为机器码占用存储空间更大，机器码比字节码大10%-20%)**
- ART的预编译机制能够改善电池续航。因为Dalvik每次运行程序都需要编译机器码，需要占用CPU资源。
- ART对垃圾回收进行了优化：1，一次GC暂停；2，GC能并行处理；3，工效高； 4，减少了后台内存使用和碎片。
### Android 7.0上的编译策略
在Android 7.0上，采用了AOT/JIT混合编译的策略。
- 应用在安装的时候dex不会被编译(JIT)；
- 应用在运行的时候，dex文件先通过解析器被直接运行，JIT会识别热点函数并存储在**jit code cache**中，然后生成profile文件来记录热点函数的信息；
- 当系统发现手机处于空闲(IDLE)或充电状态(Charging)的时候，会自动扫面App目录下的profile文件并执行AOT机制进行编译。

## 虚拟内存
### 传统内存管理存在的问题
- 一次性：作业必须一次性全部装入内存后才能开始运行。但是如果作业太大不能全部放入内存，那么该作业就不能够运行。还有就是内存不能方吓大量作业时，程序的并发度会下降。
- 驻留性：作业一旦进入内存后，就会**一直驻留在内存**中，直到作业运行结束。但是在一般情况下，只需要访问作业的部分数据，这就会导致内存资源的浪费。
### 局部性原理(虚拟内存的核心)
- 时间局部性：执行过的指令，下次执行的概率会变大；访问过的数据，下次再被访问的概率大**(程序中存在大量循环)**。
- 空间局部性：某个单元被访问后，其相邻单元被访问的几率就会变大**(很多数据在内存中时连续存放的)**。
### 虚拟内存的特性
- 多元性：作业可以被多次装入内存。
- 对换性：作业不需要常驻内存，可以根据需要换入，换出。
- 虚拟性：从逻辑上对物理内存的容量进行了扩充，使得系统认为能够使用的内存变多了。
### 页面置换算法
- 最佳置换算法(OPT)：该算法无法实现，因页面的访问序列不可知。将内存中不再使用或最长时间不再使用的页面换出。
- 先进先出置换算法(FIFO)：可能出现内存块增加，但是缺页率上升的情况(Belady现象)。
- 最近最久未使用置换算法(LRU)：当缺页中断发生的时候，选择最久未使用的哪个页面淘汰。
- 最不常用算法(LFU)：给每个页面设置一个访问计数器，当页面被访问的时候访问计数器就加1，发生缺页中断时，淘汰计数值最小的那个页面。
### LRU页面置换算法的Java实现(使用LinkedHashMap)
```java
class LRUCache extends LinkedHashMap<Integer, Integer> {
    private int capacity;
    public LRUCache(int capacity) {
        super(capacity, 0.75F, true);
        this.capacity = capacity;
    }
    public int get(int key) {
        return super.getOrDefault(key, -1);
    }
    public void put(int key, int value) {
        super.put(key, value);
    }
    @Override
    public boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest)	{
        return size() > capacity;
    }
}
```

## 数据存储
### SharedPreference
这种方式适合存储**数量少**且**格式简单**的数据。这种使用这种存储方式的数据存放在 /data/data/< package name >/shared_prefs/ 路径中。

### 文件存储
使用Java I/O流的方式。数据存放在/data/data/< package name >/files/路径下。写入数据时需要指定文件操作模式，MODE_APPEND：表示如果文件存在，则直接在文件里面追加内容；MODE_PRIVATE：表示如果文件存在，则用新的内容直接覆盖。
读取数据(简化)
```java
FileInputStream in = openFileInput("data");
BufferedReader reader = new BufferedReader(new InputStreamReader(in));
reader.readLine();
```
写入数据(简化)
```java
FileOutputStream out = openFileOutput("data", Context.MODE_PRIVATE);
BUfferedWriter writer = new BufferedWriter(new OutputStreamWriter(out));
writer.write(data);
```


### SQLite数据库
这个是专门针对嵌入式设备的轻量级嵌入式数据库引擎。能够实现类似MySQL数据库的功能。