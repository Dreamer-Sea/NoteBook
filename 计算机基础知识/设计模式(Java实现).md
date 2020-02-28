# 设计模式(Java实现)
[toc]

# 设计模式的7个原则
- 单一职责原则：一个类有且仅有一个功能(**一类功能？**)，不能够有多的功能。
- 开闭原则：对修改封闭，对扩展开放。一个对象的源码在编写完成后就不允许再迭代了，只能够基于这个对象编写新的对象的源码。
- 里氏替代原则：父类能出现的地方，子类也能够出现。该原则是对开闭原则的补充。
- 依赖倒转原则：程序要依赖于抽象(Java中的抽象类和接口)，不依赖具体的实现。从而降低用户和实现模块之间的耦合度。
- 接口隔离原则：将不同的功能定义在不同的接口中。避免类在实现接口的时候，被强迫实现不需要的功能。
- 合成/聚合复用原则：通过向新的对象中注入已有的对象，来实现类功能复用和扩展的目的。使得尽可能少的使用继承来拓展类的功能。
- 迪米特原则：尽可能的减少对象之间的依赖。使得各个模块更加独立。

# 工厂模式
在接口中定义了创建对象的方法，而将具体的对象实现放在了子类中。本质就是用工厂方法代替new操作创建一种实例化对象的方式，以提供一种方便地创建有同种类型接口的产品的复杂对象。
代码示例：
```java
//定义接口
public interface Phone{
	String brand();
}
//具体实现类
public class IPhone implements Phone{
	@Override
	public String brand(){
		return "this is a Apple phone";
	}
}
public class Huawei implements Phone{
	@Override
	public String brand(){
		return "this is a Huawei phone";
	}
}
//定义工厂类
public class Factory{
	public Phone createPhone(String phoneName){
		if ("Huawei".equals(phoneName)) return new Huawei();
		else if ("Apple".equals(phoneName)) return new IPhone();
		else return null;
	}
}
//使用工厂模式
public static void main(String[] args){
	Factory factory = new Factory;
	Phone huawei = factory.createPhone("Huawei");
	Phone iphone = factory.createPhone("Apple");
	logger.info(huawei.brand());
	logger.info(iphone.brand());
}
```

# 抽象工厂模式
由抽象接口去创建不同的工厂对象，再由工厂对象创建不同的对象。

# 单例模式
对全局可见的对象或全局变化的对象使用单例模式，保证整个程序中有且仅有该类的一个实例对象。
## 实现方式
- 懒汉模式：
```java
public class LazySingleton{
	private static LazySingleton instance;
	private LazySingleton(){}
	public static synchronized LazySingleton getInstance(){
		if (instance == null){
			instance = new LazySingleton();
		}
		return instance;
	}
}
```
- 饿汉模式：
```java
public class HungrySingleton{
	private static HungrySingleton instance = new HungtySingleton();
	private HungrySingleton(){}
	public static HungerySingleton getInstance(){
		return instance;
	} 
}
```
- 静态内部类：
```java
public class Singleton{
	private static class SingletonHolder{
		private static final Singleton INSTANCE = new Singleton();
	}
	private Singleton(){}
	public static final Singleton getInstance(){
		return SingletonHolder.INSTANCE;
	}
}
```
- 双重验证锁
```java
public class Lock2Singleton{
	private volatile static instance;
	private Lock2Singleton(){}
	public static Lock2Singleton getInstance(){
		if (instance == null){
			synchronized(Lock2Singleton.class){
				if (instance == null) instance = new Lock2Singleton();
			}
		}
		return instance;
	}
}
```

# 建造者模式：
该模式关注对象的创建过程(对象的各个部分是怎么创建的)，而工厂模式关注的是对象。
## 组成部分
- Product：要创建的对象。
- Builder：建造器，描述如何建造对象的接口。
- ConcreteBuilder：建造器的实现类，对象各个部分的具体实现。
- Director：负责使用Builder接口的对象。同一个Product虽然有相同的组件，但是组件在建造过程中的顺序可能有不同，Director对象负责安排组件的先后顺序。

# 原型模式
使用这个设计模式的类要实现Cloneable接口。有的时候程序需要以类的某个对象作为新的开始起点，而重新初始化一个对象不太方便，所以实现Cloneable接口，实现对对象的拷贝。

# 适配器模式
原始类和目标类之间存在兼容问题，通过对原始类进行扩展，构建一个适配器类，达到原始类和目标类兼容运行。当开发者想把两个类的功能结合的时候，为了减少代码的重复以及开发的可维护，就通过使用适配器类来组合两个类，使得适配器能够拥有两个类中开发者想要的功能。
## 类型
- 类适配器
- 对象适配器
- 接口适配器

# 装饰者模式
在基于‘对修改封闭，对扩展开放’的原则下，对原始类的功能进行扩展。

# 代理模式
使用构造器类，间接的使用原始类的功能。该设计模式仅改变了访问的方式，而没有改变实际的实现方式。

# 外观模式
将多个类的访问整合到一个类中，其它对象只需要通过访问这个类的对象就能够执行一系列操作，而不需要知道内部的具体实现。参考Java封装的思想。

# 桥接模式
设计一个桥接类，该类能够根据前端用户的需求调用不同的后端实现。加大了调用部分和实现部分的解耦程度。

# 组合模式
用树形结构去搭建程序。使得针对复杂对象和简单对象的操作有一致性。

# 享元模式
复用已有的资源，减少资源的创建和销毁。 

# 策略模式
对同一行为设计不同的策略，每一个策略都有自己的实现方法。

# 模板模式
将一堆类的公共方法抽取出来，单独放在一个类中，凡是需要这些公共方法的类都需要继承这个类。

# 观察者模式(发布-订阅模式)
在被观察者的状态发生变化时，系统基于事件驱动理论将其状态通知到订阅其状态的观察者对象中。
## 主要角色
- 抽象主题
- 具体主题
- 抽象观察者
- 具体观察者

# 迭代器模式
提供了顺序访问集合对象中的各种元素，而不暴露该对象内部结构的方法。

# 责任链模式
避免请求发送者与多个请求处理者耦合在一起，让一个处理者持有下一个处理者的引用，形成链状连接。

# 命令模式
将指令封装为命令，然后基于事件驱动从而异步执行。实行命令的发送者和命令的执行者解耦。

# 备忘录模式
将当前对象的内部状态保存到备忘录中。
## 主要角色
- 发起人
- 备忘录
- 状态管理者

# 状态模式
给对象定义不同的状态，每个状态有自己专属的行为。
## 主要角色
- 环境
- 抽象状态
- 具体状态

# 访问者模式
将数据结构和对数据的操作分离，使用访问者来衔接两者。

# 中介者模式
使用中介者类实现两个对象之间的交互。

# 解释器模式
给定一种语言，并定义语法格式，然后设计一个解释器来解释这个语言中的语法。

