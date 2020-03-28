# Java面试题
[toc]

# Java接口的多态性
**代码示例**
```java
interface A{
	void aa();
	void test();
}
interface B{
	void bb();
	void test();
}
class C implements A, B{
	void cc(){//code}
	@Override
	void test(){//code}
}
```
类C中的test方法重写了接口A中的test方法，还是重写了接口B中的test方法？

**解答**
可以认为类C中的test方法重写了接口A或接口B中的test方法，并不特指某一接口。
如果接口A和接口B的返回类型不同，在IDE中就会报错，编译的时候就会不通过。
如果接口A中的test方法有抛出异常，接口B中的test方法没有抛出异常，那么程序依然能够编译通过，并且正常运行。
如果接口A和接口B中的test方法都有抛出异常，但是异常不相同，那么编译能通过，运行不能通过。

**原理**
面试官提示从多态，重写和重载考虑。

# 单例模式
## 在Java下，如果破坏饿汉单例模式？
- 反射
- 序列化

## 如何保证单例模式不被破坏？
- 反射
在实现了单例模式的对象的类中声明一个私有属性。
```java
public class Singleton{
	private static volatile instance;
	private static boolean flag = false;
	
	private Singleton(){
		if (flag){
			flag = !flag;
		}else{
			try{
				throw new Exception("尝试破坏单例模式");
			}catch (Exception e){
				e.printStackTrace();
			}
		}
	}
	
	public Singleton getInstance(){
		if (instance == null){
		 synchronize(Singleton.class){
		 	if (instance == null){
		 		instance = new Singleton();
		 	}
		 }
		}
		return instance;
	}
}
```

- 序列化
在实现单例模式的类中加入以下的方法：
```java
private Object readResolve() {
	return instance;
}
```

# 内部类
## 内部类的属性
不论内部类的属性是否是静态，使用的修饰符是哪个，外部类都能够获得内部类的属性。
## 内部类方法
不论内部类的方式是否是静态的，使用的修饰符是哪个，外部类都能够获得内部类的方法。

# 内存泄漏
内存泄漏指的是无用对象持续占有内存或者无用对象的内存得不到及时释放，从而造成内存空间的浪费。
## 造成原因
1. 静态集合类引起的。
2. 集合里面的对象属性被修改后，再调用remove方法时不起作用。
3. 释放对象的时候没有删除监听器。
4. 各种连接在不使用的时候，没有及时关闭。
5. 非静态内部类持有外部类引用。
6. 单例模式。

# 反射是如何实现的
所有类的Class对象都会在JVM运行的时候在**运行时方法区(堆的一部分)**中生成。这个生成过程需要经历**类加载**该过程(加载-(验证-准备-解析)-初始化)。在使用反射方式实例化类的时候需要经历**对象创建**的过程。
## 对象创建的过程
1. 类加载检查。
2. 分配内存。
3. 初始化零值。
4. 设置对象头。
5. 执行init方法。

# 子类和父类的关系
1. 子类不能获得父类的私有属性和私有方法。
2. 子类在重写的时候，访问符的范围不能小于父类的访问符的范围。

# synchronized关键字和ReentrantLock的区别
1. 两者都是可重入锁。
2. 前者依赖JVM，后者依赖于API。
3. 后者支持的功能比前者多，如：非公平锁/公平锁、中断机制。
4. 前者使用wait/notify/notifyAll实现等待/通知机制，后者使用newCondition/signal/signalAll实现等待/通知机制。

# GC的触发条件
## Minor GC的触发条件
当Eden区满时，触发Minor GC。
## Full GC的触发条件
1. 调用System.gc时，可能触发。
2. 老年代空间不足。
3. 方法区空间不足。
4. 通过Minor GC后进入老年代的平均大小**大于**老年代的可用内存。
5. Minor GC时，对象大小**大于**To区的可用内存时，且该对象大小**大于**老年代的可用内存大小。

# 锁优化和膨胀过程
1. 自旋锁
2. 锁粗化
3. 锁销除
4. 偏向锁
5. 轻量级锁
6. 重量级锁

# 双亲委派模型
## 如何破坏双亲委派模型
1. 继承ClassLoader。
2. 重写findClass方法。
3. 重写loadClass方法。

## 双亲委派模型的优点
1. 避免类的重复加载。
2. 保护核心API。

# 泛型
## 上下界
1. 上界
```java
<? extends T>
```
2. 下界
```java
<? super T>
```
## 泛型擦除
因为Java是**伪泛型**的，在编译阶段会将泛型信息给擦除。

# Java中的堆是线程共享的吗？
不是。
在线程中创建了新的对象就需要在堆上分配内存。在多线程的情况下，可能出现线程竞争的问题。为了避免这个问题，在每个线程创建的时候，HotSpot虚拟机会给线程分配一定的堆内存，用来放置线程创建的对象的(TLAB，Thread Local Allocation Buffer)。这里仅仅是分配内存(**操作**)不共享，但是这块区域每个线程都能够访问到。

# 线程池中的队列如何保证线程安全
队列中负责提交任务和消费任务的方法中，使用了ReentrantLock这个类来实现线程间的同步。能够避免任务提交时带来的覆盖问题。