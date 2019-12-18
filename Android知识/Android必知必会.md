# Android必知必会
[toc]

## Application的生命周期
- onCreate()：程序创建的时候运行。
- onTerminate()：程序终止的时候运行
- onLowMemory()：低内存的时候执行。
- onTrimMemory()：在内存清理的时候执行。
### onConfigurationChanged方法
该方法在Application/Activity/Fragment中都有。在系统配置发生变化的时候就能够调用这个方法做出相应的处理，需要在AndroidManifest.xml中配置。例如当屏幕翻转的时候，可以不用重建Activity。

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

## Dalvik虚拟机和标准Java虚拟机的区别
- Dalvik基于寄存器，JVM基于栈。基于寄存器的虚拟机对于更大的程序来说，它们在编译的时候花费的时间更短。
- 在JVM字节码中，局部变量会被放入局部变量表中，继而被压入堆栈供操作码进行运算，当然JVM也可以只是用堆栈而不显式地将局部变量存入变量表中。
- 在Dalvik字节码中，局部变量会被赋给**65536**个可用地寄存器中地任何一个，Dalvik指令直接操作这些寄存器，而不是访问堆栈中地元素。
- JVM运行的是Java字节码，Dalvik运行的是自定义的dex字节码格式。
- Dalvik的常量池被修改为只使用32位的索引。
- 在Android，一个应用，一个Dalvik虚拟机实例，一个Linux进程是对应的。

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
这种方式适合存储**数量少**且**格式简单**的数据。这种使用这种存储方式的数据存放在 /data/data/< package name >/shared_prefs/ 路径中。存储格式为xml。
写入数据
```java
SharedPreferences.Editor editor = getSharedPreferences("data", MODE_PRIVATE).edit();
editor.putString("name", "Tom");
editor.putInt("age", 28);
editor.putBoolean("married", false);
editor.apply();
```
读取数据
```java
SharedPreference pref = getSharedPreference("data", MODE_PRIVATE);
String name = pref.getString("name", "");
int age = pref.getInt("age", 0);
boolean married = pref.getBoolean("married", false);
```
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

## Activity的启动模式
Activity的启动模式一共有四种，分别是：standard(默认的)，singleTop，singleTask，singleInstance。
- standard：这是默认的启动模式，每次启动Activity都能创建一个该Activity的新的实例。
- singleTop：如果该Activity位于任务栈的栈顶，则不创建新的实例，如果不位于栈顶则创建新的实例。该启动模式适合接收通知启动的内容显示页面。
- singleTask：如果任务栈中不存在Activity，则创建一个新的实例，如果任务栈中存在该Activity，则将该Activity之前的Activity全部出栈，使得该Activity位于栈顶。适合作为程序的入口，例如浏览器的主界面。
- singleInstance：为该Activity创建一个单独的任务栈。适合需要与主程序分离的页面。例如闹钟提醒，将闹钟提醒与闹钟设置分离。

## Application Not Responding(ANR)
即应用程序无响应。一般这种情况下，系统会弹出一个对话框问用户是要继续等待应用程序还是要立马终止应用程序。
### 触发条件
- InputDispatching Timeout：**5秒**内无法响应屏幕触摸事件或键盘输入事件。
- BroadcastQueue Timeout：在执行前台广播(BroadcastReceiver)的**onReceive()**函数时，**10秒**没处理完成，后台为**60秒**。
- Service Timeout：前台服务**20秒内**，后台服务**200秒内**没有执行完。
- ContentProvider Timeout：ContentProvider的publish在**10秒内**没进行完。
### 如何避免
将可能阻塞UI线程的操作，如网络请求，Socket通信，SQL操作，文件读写等放在子线程中。
特别是在Activity生命周期地关键方法**onCreate()和onResume()**中应该尽可能做比较少的事情。
建议使用AsyncTask执行耗时的方法。

## Fragment
### 生命周期
- onAttach()：Fragment和Activity相关联时调用。可以通过该方法获取Activity引用，还可以通过getArguments()获取参数。
- onCreate()：Fragment被创建时调用。
- onCreateView()：创建Fragment布局。
- onActivityCreated()：当Activity完成onCreate()时调用。
- onStart()：当Fragment可见时调用。
- onResume()：当Fragment可见且可交互时调用。
- onPause()：当Fragment不可交互但可见时调用。
- onStop()：当Fragment不可见时调用。
- onDestoryView()：当Fragment的UI从视图结构中移除时调用。
- onDestory()：销毁Fragment时调用。
- onDetach()：当Fragment和Activity解除关联时调用。

## ListView的优化方式
### convertView的复用
ListView的getView方法中有一个convertView参数，通过判断该参数是否为null，如果为空则创建一个视图，然后给视图设置数据，最后将这个视图返回给底层；如果不为null的化，其他新的view可以通过复用的方式使用已经消失的条目view，重新设置上数据然后展现出来。
```java
public View getView(int position, View convertView, ViewGroup parent) {
    if (convertView == null) {
	//如果当前的convertView为null，则通过inflate产生一个view
    convertView = View.inflate(context, R.layout.layout_pic_item,null);
	}
	TextView tvDis = (TextView) convertView.findViewById(R.id.tv_item_picture_desc);
    tvDis.setText("设置数据");
    return convertView;
}
```
### 使用ViewHolder
findViewById方法也会影响加载的速度，所以可以在ListView的内部构造一个ViewHolder内部类，这个ViewHolder包含了这个View的所有组件。当发现convertView不为空的时候，直接用getTag方法从convertView中取出ViewHolder。
```java
@Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder holder;
        View itemView = null;
        if (convertView == null) {
            itemView = View.inflate(context, R.layout.item_news_data, null);
            holder = new ViewHolder(itemView);
            //用setTag的方法把ViewHolder与convertView "绑定"在一起
            itemView.setTag(holder);
        } else {

    //当不为null时，我们让itemView=converView，用getTag方法取出这个itemView对应的holder对象，就可以获取这个itemView对象中的组件

            itemView = convertView;
            holder = (ViewHolder) itemView.getTag();
        }

        NewsBean newsBean = newsListDatas.get(position);
        holder.tvNewsTitle.setText(newsBean.title);
        holder.tvNewsDate.setText(newsBean.pubdate);
        mBitmapUtils.display(holder.ivNewsIcon, newsBean.listimage);

        return itemView;
    }

}

public class ViewHolder {
    @ViewInject(R.id.iv_item_news_icon)
    private ImageView ivNewsIcon;// 新闻图片
    @ViewInject(R.id.tv_item_news_title)
    private TextView tvNewsTitle;// 新闻标题
    @ViewInject(R.id.tv_item_news_pubdate)
    private TextView tvNewsDate;// 新闻发布时间
    @ViewInject(R.id.tv_comment_count)
    private TextView tvCommentIcon;// 新闻评论

    public ViewHolder(View itemView) {
        ViewUtils.inject(this, itemView);
    }
}
```
### 分段加载
当数据量较大时，将数据分段加载这样能够避免花大量的时间来一次性加载太多数据。
### 分页加载
上一种方法在加载下一段数据的时候并不会将List清空，如果数据特别多那么List就会变得特别大，从而导致OOM。分页加载能够将之前加载的内容从List中移除，放入新的内容。

## Service
Service在启动后并不会运行在单独的进程或线程中，它依旧是在主线程中运行。
按不同类型分类：
- 运行地点：本地服务；远程服务。
- 运行类型：前台服务；后台服务。
- 功能：可通信服务；不可通信服务。
### 生命周期涉及的方法
自动调用：
- onCreate()：创建服务。
- onStartCommand()：开始服务。
- onDestroy()：销毁服务。
- onBind()：绑定服务。
- onUnbind()：解绑服务。
手动调用：
- startService()：启动服务。
- stopService()：关闭服务。
- bindService()：绑定服务。
- unbindService()：解绑服务。
startService()/bindService()这两个方法都能够启动服务，但是后者能够使得Activity和Service进行更多的交互。
### IntentService
Service时运行在主线程中的，关闭或者解绑都需要显式的调用相应的方法。而IntentService是会专门创建一个工作线程来处理任务(一般是多线程任务)，任务完成后IntentService就会自动关闭。
IntentService继承于Service，并给onBind()方法和onStartCommand()方法提供了默认的实现。前者是返回null，后者是将请求的intent添加到队列中。
相比起普通的线程，IntentService的优先级较高。
### 优先级
使用后台服务的时候，如果发生了内存不足的情况，后台服务可能被回收，因为它的优先级比较低。但是使用前台服务(通知+服务)就能够提高服务的优先级。

## 内存管理
### 垃圾回收
与Java的GC类似。
### 共享内存
Android可以跨进程共享RAM页面(Pages)。
- 所有应用程序进程都是从Zygote的现有进程分叉(fork)出来的。
### 分配机制
Android在为每个进程分配内存时，采用**弹性**的分配方式，即刚开始的时候并不会给应用分配很多的内存，而是给每个进程分配一个“够用”的内存大小。这个大小值是根据每一个设备的实际的物理内存大小来决定。随着应用的运行和使用，Android会为进程分配一些额外的内存。但是分配的大小是有限度的。Android需要最大限度的让更多的进程存货在内存中，以保证用户再次打开应用时减少应用的启动时间，提高用户体验。
### 回收机制
Android会根据以下两点来判断是否应该回收进程所占用的内存。
第一是进程优先级，从高到低分别为：
- 前台进程
- 可见进程
- 服务进程
- 后台进程
- 空进程
第二是回收收益