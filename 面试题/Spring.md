# Spring

[toc]

# Spring中使用了哪些设计模式
## 简单工厂
比如：BeanFactory。
意义：松耦合；bean的额外处理。

## 工厂方法
比如：FactoryBean。

## 单例模式
比如：Spring依赖注入Bean实例。

## 适配器模式
比如：HandlerAdatper。

## 装饰器模式：
比如：类名中含有Wrapper或Decorator的。

## 代理模式
比如：AOP底层，使用动态代理模式的实现。

## 观察者模式
比如：Spring中的事件驱动模型。

## 策略模式
比如：资源访问Resource接口。

## 模板方法模式
比如：JDBC的抽象和对Hibernate集成。

# 什么是Spring Bean？
1. Bean是由Spring IoC容器实例化、配置、装配和管理。
2. Bean是基于用户提供给IoC容器的配置元数据Bean Definition创建。

# Spring如何解决循环依赖的问题？
Spring中对Bean是实例化是采用循环递归的方式，首先创建对象，再为其注入属性。即Spring在实例化一个bean的时候，首先递归实例化其所依赖的所有bean，知道某个bean没有依赖其它bean，此时就会将该实例返回，然后反递归的将获取到的bean设置位各个上层bean的属性。


