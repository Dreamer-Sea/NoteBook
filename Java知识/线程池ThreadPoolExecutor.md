# 线程池
[toc]

## 阻塞队列
### ArrayBlockingQueue
基于数组实现的有界阻塞队列。按照先进先出的顺序对元素进行排序。

### LinkedBlockingQueue
基于链表实现的有界阻塞队列。对生产者端和消费者端分别采用了两个独立的锁来控制数据同步。

### PriorityBlockingQueue
按优先级排序的无界阻塞队列。

### DelayQueue
支持延时获取元素的阻塞队列。基于优先级队列实现的无界阻塞队列。

### SynchronousQueue
用于控制互斥操作的阻塞队列。是一个不存储元素的阻塞队列。

### LinkedTransferQueue
基于链表结构实现的无界阻塞队列。

### LinkedBlockingDeque
基于链表结构实现的双向阻塞队列。

## 线程池参数
- corePoolSize：线程池核心线程个数。
- workQueue：用于保存等待执行的任务的阻塞队列，例如ArrayBlockingQueue(基于数组的有界队列)，LinkedBlockingQueue(基于链表的无界队列)，SynchronousQueue(最多只有一个元素的同步队列)以及PriorityBlockingQueue(优先级队列)。
- maximumPoolSize：线程池最大线程数量。
- ThreadFactory：创建线程的工厂(一般默认就行)。
- RejectedExecutionHandler： 饱和策略，当等待队列满了且线程个数达到maximumPoolSIze后采取的策略。
	- AbortPolicy(直接抛出异常)；
	- CallerRunsPolicy(由调用提交任务的线程来执行这个任务 )；
	- DiscardOldestPolicy(调用poll丢弃一个任务，执行当前任务)；
	- DiscardPolicy(默默丢弃，不抛出异常)。
- keepAliveTime：非核心线程的存活时间。
- TimeUnit：存活时间的时间单位。

## 线程池类型
- newFixedThreadPool：核心线程数和最大线程数一致，阻塞队列长度为Integer.MAX_VALUE，keepAliveTIme=0的线程池(非核心线程只要空闲就立马回收)。
- newSingleThreadExecutor：核心线程数和最大线程数为1，阻塞队列长度为Integer.MAX_VALUE，keepAliveTime=0。
- newCachedThreadPool：创建一个按需创建线程的线程池，初始线程个数为0，最多线程个数为Integer.MAX_VALUE，且阻塞队列为同步队列，keepAliveTime=60。加入同步队列的任务会马上执行，同步队列最多只有一个任务。
- ScheduledThreadPoolExecutor：线程池中的任务可定期执行，或延期执行。
- newWorkStealingPool：JDK 1.8新增。由JVM自行判断需要的线程数。

## 停止线程池的方法
- shutdown：发出停止指令，线程池不再接收新的任务，线程池在等待队列中的任务全部执行完后就会关闭。
- shutdownNow：线程池立即停止，当前执行的任务全部抛弃，等待队列中的任务也全部抛弃。

# ScheduledThreadPoolExecutor
这个线程池能够实现在指定的时间后或者定时执行任务调度。
## 任务提交的方法
- schedule(Runnable command, long delay, TimeUnit unit)：提交的任务在delay时间后执行。
- scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit)：提交的任务在第一次执行完后，在delay时间后在执行。
- scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)：周期性执行提交的任务。

## 自定义拒绝策略
实现RejectedExecutionHandler，并实现rejectedExecution方法。
