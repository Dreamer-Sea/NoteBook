# 面试题
[toc]
## Java
### Java接口的多态性
#### 代码示例
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
#### 解答
可以认为类C中的test方法重写了接口A或接口B中的test方法，并不特指某一接口。
如果接口A和接口B的返回类型不同，在IDE中就会报错，编译的时候就会不通过。
如果接口A中的test方法有抛出异常，接口B中的test方法没有抛出异常，那么程序依然能够编译通过，并且正常运行。
如果接口A和接口B中的test方法都有抛出异常，但是异常不相同，那么编译能通过，运行不能通过。
#### 原理
面试官提示从多态，重写和重载考虑。

