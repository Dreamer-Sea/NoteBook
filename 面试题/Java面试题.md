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