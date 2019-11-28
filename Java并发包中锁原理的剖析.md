# Java并发包中锁原理的剖析
[toc]
## LockSupport类
LockSupport类与每个使用它的线程都会关联一个许可证，在默认情况下调用LockSupport类的方法的线程是不持有许可证的。LockSupport类是使用Unsafe类实现的。
### LockSupport类中主要的函数
#### void park()方法
如果调用park方法的线程已经拿到了与LockSupport类关联的许可证，则在调用park方法的时候会立马返回，否则线程就会被阻塞挂起。
```java
public static void main(String[] args){
    System.out.println("begin park!");
    LockSupport.park();
    System.out.println("end park!");
}
```
当其他线程调用unpark(Thread thread)方法并将当前线程作为参数时，调用park方法被阻塞的线程会立马返回。
此外，如果有其它线程调用了被park方法阻塞线程的interrupt方法，设置了中断标志或者线程被虚假唤醒，那么线程也会返回，并且不会抛出InterruptedException异常。
#### void unpark(Thread thread)方法
当一个线程调用unpark方法的时候，如果thread线程没有持有thread与LockSupport类关联的许可证，则thread线程会立马获得许可证。
如果thread线程在这之前被park方法挂起，则调用unpark方法后，thread线程会被立马唤醒。
如果thread之前没有调用过park方法，则调用unpark方法后，再调用park方法，该线程就会被立即返回。
#### void parkNanos(long nanos)方法
该方法与park方法类似，但是加入了线程挂起的时间，当线程挂起nanos时间后就会自动返回。
#### void park(Object blocker)方法
当线程因为没有许可证而被阻塞挂起时，这个blocker对象会被记录到该线程的内部，使用诊断工具通过调用getBlocker(Thread)方法能够获取这个blocker对象。这样再打印线程堆栈的时候，就能够知道是哪个类被阻塞了。

## AbstractQueuedSynchronizer抽象同步队列(AQS)
Java并发包(JUC)中的锁的底层实现就是AQS。AQS是一个双向队列，节点内部分别用SHARED/EXCLUSIVE来表示节点是因为获取共享资源失败还是获取独占资源失败而被放入队列的。waitStatus记录当前线程的等待状态，可以为CANCELLED(线程被取消了)，SINGAL(线程需要被唤醒)，CONDITION(线程在条件队列里面等待)，PROPAGATE(释放共享资源的时候需要通知其它节点)。
### state变量
AQS中有一个很重要的变量state，它可以通过getState，setState和compareAndSetState被修改。在ReentrantLock中，state用来表示当前线程获取锁的可重入次数；对于读写锁ReentrantReadWriteLock来说，state的高16位表示读状态，也就是获得该读锁的次数，低16位表示获取到写锁的线程的可重入次数；对于Semaphore来说，state用来表示当前可用信号的个数；对于CountDownLatch来说，state用来表示计数器的当前值。
### 独占方式下，资源的获取与释放：
- 获取：当一个线程调用acquire(int arg)方法获取独占资源的时候，会先使用tryAcquire方法尝试获取资源，即设置state的值，如果成功就返回，不成功就将当前线程封装为类型位Node.EXCLUSIVE的节点，然后将该节点插入到AQS阻塞队列的尾部，并调用LockSupport.park(this)方法挂起自己。  
- 释放：当一个线程调用release(int arg)方法时，会先尝试使用tryRelease操作释放资源，同样是设置state的值，然后调用LockSupport.unpark(thread)方法激活AQS队列里面被阻塞的一个线程。被激活的线程会调用tryAcquire方法查看当前state的值能否满足自己的需要(**满足什么需要？**)，满足则该线程被激活，并继续向下运行，否则还是会被放入AQS队列并被挂起。
### 共享方式下，资源的获取与释放：
在共享方式下，资源的获取与释放同独占方式下的类似。
### 条件变量的支持
用AQS实现的锁能够使用条件变量的signal和await方法实现，类似synchronized+notify+wait的功能。不同的时synchronized同时只能与一个共享变量的notify或wait方法实现同步，而AQS的一个锁可以对应多个条件变量。
与notify+wait方法一样，调用条件变量的signal和await方法时也需要先获取条件变量对应的锁。

## ReentrantLock
可以通过传入的参数设置该锁为公平锁或非公平锁，默认是非公平锁。
### 获取锁的方法：void lock()
该锁用void lock()方法获取锁，如果该锁未被其它线程持有，就将该锁的持有者设置为当下调用lock方法的线程，并将state设置为1；如果当前锁不空闲，但是持有锁的线程和当下调用lock方法的线程一致，则将state+1；如果试图获取锁的线程与持有锁的线程不一致，就将该线程放入AQS队列并阻塞挂起。
### 公平锁的实现
该锁的非公平锁和公平锁的实现区别仅在于，公平锁的实现中会判断阻塞队列是否为空，空的话才将当前线程设置为锁的持有者，不为空则由AQS队列的头节点持有锁。
### 该锁能够响应中断
### 释放锁的方法：void unlock()
如果调用该方法的线程持有锁，则将state-1；如果state = 0，则释放锁，否则继续运行；如果调用线程不持有锁，则抛出IllegalMonitorStateException异常。

## ReentrantReadWriteLock
ReentrantLock不论是读操作还是写操作都会加锁，但是读操作其实并不需要加锁，所以在读操作比较多的时候，不断的加锁解锁会造成不必要的CPU资源浪费，所以基于AQS构造了ReentrantReadWriteLock。该锁能够做到读写分离，允许多个线程获得读锁。
### 关键实现点
该锁将state分为高16位和低16位，高16位表示读状态，即获取到读锁的次数；低16位表示获取到写锁的线程的可重入次数。
### 读写锁
该锁的读写锁存在一定的关联。如果写锁不被持有，则线程能够获得读锁，如果写锁被持有，则线程不能够获得读锁。


## StampedLock
该锁是JDK 8新提供的。
它提供了三种模式的读写控制：
- 写锁：写锁请求成功后会返回一个stamp变量来表示该锁的版本。释放写锁的时候需要将这个stamp作为参数传入。不可重入。
- 悲观读锁：写锁空闲时，该锁才能被获取。获取读锁成功后会返回一个stamp变量，在解锁的时候需要传入这个stamp变量。
- 乐观读锁：用为预算的方法测试该锁是否空闲。
这三种锁能在一定条件下进行转换。