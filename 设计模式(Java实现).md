# 设计模式(Java实现)
[toc]
## 设计模式的7个原则
- 单一职责原则：一个类有且仅有一个功能(**一类功能？**)，不能够有多的功能。
- 开闭原则：对修改封闭，对扩展开放。一个对象的源码在编写完成后就不允许再迭代了，只能够基于这个对象编写新的对象的源码。
- 里氏替代原则：父类能出现的地方，子类也能够出现。该原则是对开闭原则的补充。
- 依赖倒转原则：程序要依赖于抽象(Java中的抽象类和接口)，不依赖具体的实现。从而降低用户和实现模块之间的耦合度。
- 接口隔离原则：将不同的功能定义在不同的接口中。避免类在实现接口的时候，被强迫实现不需要的功能。
- 合成/聚合复用原则：通过向新的对象中注入已有的对象，来实现类功能复用和扩展的目的。使得尽可能少的使用继承来拓展类的功能。
- 迪米特原则：尽可能的减少对象之间的依赖。使得各个模块更加独立。

## 设计模式的分类

## 工厂模式
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
