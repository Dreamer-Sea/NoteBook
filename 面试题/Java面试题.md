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

