# Java多线程
[toc]

# 进程和线程的概念
进程是代码在数据集合上的一次运行，是系统进行资源分配和调度的基本单位，线程则是进程的一个执行路径，一个进程中最少有一个线程，进程中多个线程共享进程的资源。

# Java中使用的线程调度算法
目前常见的线程调度算法有两种：分时调度模型和抢占式调度模型。Java中使用的是后者。

# 线程饥饿
线程饥饿指的是：一个或多个线程因为种种原因而无法获得所需的资源，导致一直无法执行的状态。
**原因**
1. 高优先级线程吞噬所有的低优先级线程的CPU时间。
2. 线程被永久堵塞在一个等待进入同步块的状态，因为其他线程总是能在它之前持续地对该同步块进行访问。
3. 线程在等待一个本身也处于永久等待完成的对象，因为其他线程总是被持续地获得唤醒。

# 线程的生命周期
- NEW：刚创建。
- RUNNABLE：可运行，等待CPU时间片。
- BLOCKED：被阻塞挂起。
- WAITING：线程调用了这些方法：Object.wait，Thread.join，LockSupport.park。
- TIMED_WAITING：调用了前者那些方法的限时版。
- TERMINATED：死亡(销毁)。
## 状态之间的转移
- Running -> Runnable：调用yield方法，在执行完任务后立马释放CPU资源。
- Running -> Dead：
	- run/call方法执行完毕。
	- Exception。
	- Error。
	- 调用Stop方法。
- Running -> Blocked：
	- 调用Sleep方法。
	- I/O阻塞。
	- 等待同步锁。
	- 等待通知。
	- 调用suspend方法。
- Blocked -> Runnable：
	- sleep时间到。
	- I/O方法返回。
	- 获得同步锁。
	- 收到通知。
	- 线程resume。

# 构建多线程的四种方式
- 继承Thread类
- 实现Runnable接口
- 实现Callable接口
- 使用线程池

## 继承Thread类
```java
public class MyThread extends Thread{
	@Override
	public void run(){
		System.out.println(Thread.currentThread() + " 正在运行");
	}
}
```
## 实现Runnable接口
```java
public class MyRunnable implements Runnable{
	@Override
	public void run(){
		System.out.println(Thread.currentThread() + " 正在运行");
	}
}
```

## 实现Callable接口
```java
public class MyCallable implements Callable{
    @Override
    public String call() throws Exception {
        return Thread.currentThread() + "";
    }
}

public static void main(String[] args)  {
        FutureTask<String> futureTask = new FutureTask<>(new MyCallable());
        Thread t1 = new Thread(futureTask);
        t1.start();
        try{
            String result = futureTask.get();
            System.out.println(result + " 正在运行");
        } catch (ExecutionException | InterruptedException e) {
            e.printStackTrace();
        }
    }
```
## 实现Runnable接口和Callable的区别
实现Callable接口能够有返回值和抛出异常，这些都是Runnable接口都没有的。

# 线程通知与等待
1. wait()方法

当一个线程调用一个共享变量的wait()方法时，该调用线程会被阻塞挂起，直到发生下面几件事情之一才返回：
- 其它线程调用了该共享对象的notify()方法或者notifyAll()方法；
- 其它线程调用了该线程的interrupt()方法，该线程抛出InterruptedException异常。

注意：如果调用wait()方法的线程没有事先获取该对象的监视器锁，则调用wait()方法时会抛出IllegalMonitorStateException异常。

线程如何获取一个共享变量的监视器锁：
(1)执行synchronized同步代码块时，使用该共享变量作为参数。
```java
synchronized(共享变量){
	//doSomething
}
```
(2)调用该共享变量的方法，并且该方法使用了synchronized修饰。
```java
synchronized void add(int a, int b){
	// doSomething
}
```
线程被挂起后，可能自动从挂起状态转为可运行状态，这就是虚假唤醒。
解决方法：不断测试线程的唤醒条件是否满足，不满足就继续等待。
```java
synchronized (obj){
	while (条件不满足){
		obj.wait();
	}
}
```
当前线程调用wait()方法只会释放当前共享变量上的锁，不影响当前线程上其它共享变量的锁。

wait(long timeout)方法：
能给wait方法设置超时参数，如果线程在被挂起后没有在timeout的时间内被唤醒，那该方法会因为超时而返回。

2. notify()方法
一个线程在调用了共享变量的notify()方法后，会随机唤醒一个在该共享变量上因调用wait系列方法而被挂起的线程。唤醒后的线程会变为就绪状态，只有在线程获得了共享变量的监视器锁才能继续执行。
注意：只有当前线程获得了共享变量的监视器锁后，才可以调用共享变量的notify()方法，否则会抛出IllegalMonitorStateException异常。

3. notifyAll()方法
该方法会唤醒共享变量上所有因调用了wait系列方法而挂起的线程。

# 等待线程执行终止的join方法
```java
public static void main(String[] args) throws InterruptedException{
	Thread threadOne = new Thread(new Runnable(){
		@Override
		public void run(){
			try{
				Thread.sleep(1000);
			}catch(InterruptedException e){
				e.printStackTrace();
			}
			System.out.println("child threadOne over!");
		}
	});
	
	Thread threadTwo = new Thread(new Runnable(){
		@Override
		public void run(){
			try{
				Thread.sleep(1000);
			}catch(InterruptedException e){
				e.printStackTrace();
			}
			System.out.println("child threadTwo over!");
		}
	});
	
	threadOne.start();
	threadTwo.start();
	
	System.out.println("wait all child thread over!");
	
	//等待子线程执行完毕，返回
	threadOne.join();
	threadTwo.join();
	
	System.out.println("all child thread over!");
}
```

# sleep()方法让线程睡眠
一个在运行中的线程能够调用Thread类中的sleep方法，使得该线程让出一定时间的执行权，但是该线程不会释放所持有的监视器锁。当时间到了后，线程就能回到就绪状态参与CPU的调度。如果在睡眠的过程中，有其它线程调用了该线程的interrupt()方法来中断该线程，那么该线程就会在调用sleep方法的地方抛出InterruptedException异常并返回。

# sleep方法和wait方法的区别
- sleep方法属于Thread类，wait方法属于Object类。
- sleep方法能持有锁，wait方法释放锁。
- sleep方法仅让出CPU时间，在设定的时间到了后会立马恢复到Runnable状态。

# yield()方法让线程让出CPU使用权
该方法使得线程让出CPU使用权，并将自身变为就绪状态，而不是被阻塞挂起。如果该线程在让出了CPU使用权后，又立马获得了CPU的使用权，那么它会直接运行。

# 线程中断
## void interrupt()方法
当线程A在运行的时候，如果线程B可以调用线程A的interrupt()方法来设置线程A的中断标志为true并立即返回。这里线程B仅仅是设置了中断标志，并不是中断线程A，实际上线程A还在继续运行中。
如果线程A因为调用了wait系列方法，join方法或者sleep方法而被阻塞挂起，这时候如果线程B调用线程A的interrupt()方法，线程A会在调用这些方法的地方抛出InterruptedException异常而返回。
## boolean isInterrupted()方法
检测当前线程是否被中断，如果是返回true，否则返回false。
## boolean interrupted()方法：
检测当前线程是否被中断，是的话返回true，否的话返回false。
该方法如果发现当前线程被中断，则会清除中断标志。该方法是获取当前调用线程的中断标志而不是调用interrupted()方法的实例对象的中断标志。

# 线程的终止
- 正常运行结束。
- 使用退出标志。
- 使用Interrupt方法。
- 使用stop方法(已被抛弃)。

# 线程的上下文切换
CPU资源的分配是按照时间轮转的方法来的，所以当一个线程的时间片用完之后，就要让出CPU资源，变为就绪状态，与其它就绪状态的线程竞争CPU资源(获得时间片)。让出CPU资源的过程就叫做上下文切换。
线程上下文 切换的时机：
- 当前线程的时间片使用完了，并处于就绪状态；
- 当前线程被其它线程中断。

# 守护线程和用户线程
Java中的线程分为两类，分别是守护线程(daemon thread)和用户线程(user thread)。在JVM启动时会调用main函数，main函数就是一个用户线程，同时还会启动多个守护线程(例如垃圾回收线程)。
**区别**：当最后一个用户线程结束的时候，JVM就会正常退出，不论是否有守护线程在运行。
如何创建守护线程？
```java
public static void main(String[] args){
	Thread daemonThread = new Thread(new Runnable(){
		public void run(){
			
		}
	});
}
daemonThread.setDaemon(true);
daemonThread.start();
```

# ThreadLocal
ThreadLocal变量使得每个访问这个变量的线程都能够有一份独有的该变量的本地副本。不同线程在操作这个变量的时候，实际上是在操作自己本地内存中的变量。这样就能够避免线程安全的问题。
代码示例
```java
public class ThreadLocalTest {

    static ThreadLocal<String> localVariable = new ThreadLocal<>();

    public static void main(String[] args) {
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                localVariable.set("t1 local variable");
                print("t1");
                System.out.println("t1 remove after:" + localVariable.get());
            }
        });
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                localVariable.set("t2 local variable");
                print("t2");
                System.out.println("t2 remove after:" + localVariable.get());
            }
        });

        t1.start();
        t2.start();
    }

    public static void print(String str){
        System.out.println(str + ":" + localVariable.get());
        localVariable.remove();
    }
}
```
默认情况下Thread类中的threadLocals变量和inheritableThreadLocals变量都为null，只有当前线程第一次调用ThreadLocal的set或者get方法时才会创建它们。这两个变量都时定制化的HashMap。

默认情况下ThreadLocal变量不支持继承，即子线程不能获得父线程的本地变量。如果想要子线程能够获得父线程的本地变量就要使用InheritableThreadLocal类，子线程获得是父线程本地变量的副本。

# 线程死锁
死锁产生的四个必备条件：
- 互斥条件：同一时间，一个资源只能够被一个线程获得；
- 请求与保持条件：一个线程在获得了一个资源之后，在完成任务前都不会释放这个资源；
- 不可剥夺条件：一个线程获得的资源不被另一个线程抢占，只能够自己释放资源；
- 环路等待条件：例如线程1获得了资源A，线程2获得了资源B，现在线程1在等待资源B，线程2在等待资源A。
代码示例：
```java
public static void main(String[] args) throws InterruptedException {

        Object resA = new Object();
        Object resB = new Object();

        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (resA){
                    try {
                        System.out.println("Thread1 has the resA");
                        Thread.sleep(1000);
                        synchronized (resB){
                            System.out.println("Thread2 has the resB");
                        }
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        });

        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (resB){
                    System.out.println("Thread2 has the resB");
                    synchronized (resA){
                        System.out.println("Thread1 has the resA");
                    }
                }
            }
        });

        t1.start();
        t2.start();
    }
```
以上4个条件中只有请求保持和环路等待是可以被破坏的。

# 线程锁死(活锁)
线程所需的条件永远无法成立，或者其它线程无法唤醒这个线程，从而使它一致处于非运行状态。
## 类型
- 信号丢失锁死：没有对应的通知来将等待的线程唤醒。
- 嵌套监视器锁死：内层锁被唤醒(Y.wait)，但是外层锁没被唤醒，但是通知线程没有获得外层锁X，所以不能用Y.notifyAll来唤醒等待线程。
