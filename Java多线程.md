# Java多线程
[toc]

## 进程和线程的概念
进程是代码在数据集合上的一次运行，是系统进行资源分配和调度的基本单位，线程则是进程的一个执行路径，一个进程中最少有一个线程，进程中多个线程共享进程的资源。

## 程序计数器
每一个线程都有一个独占的程序计数器，负责记录线程当前要执行的指令地址。在多线程中，主要用来记录线程上次让出CPU时，程序执行到了哪里，便于下次获得CPU时间片后

## 构建多线程的三种方式
* 继承Thread类
* 实现Runnable接口
* 实现Callable接口

### 继承Thread类
```java
public class MyThread extends Thread{
	@Override
	public void run(){
		System.out.println(Thread.currentThread() + " 正在运行");
	}
}
```
### 实现Runnable接口
```java
public class MyRunnable implements Runnable{
	@Override
	public void run(){
		System.out.println(Thread.currentThread() + " 正在运行");
	}
}
```

### 实现Callable接口
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

## 线程通知与等待
1. wait()方法

当一个线程调用一个共享变量的wait()方法时，该调用线程会被阻塞挂起，直到发生下面几件事情之一才返回：
	(1)其它线程调用了该共享对象的notify()方法或者notifyAll()方法；
	(2)其它线程调用了该线程的interrupt()方法，该线程抛出InterruptedException异常。

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

2. notify()方法
3. notifyAll()方法

