# Spring框架
[toc]
## 什么是控制反转(IoC)
将对象间的依赖关系交给Spring容器来处理，使用配置文件来创建所依赖的对象，由**主动**创建对象变为**被动**创建对象，实现**解耦合**。使用@Autowired和@Resource来注入对象，被注入的对象必须被以下四个注解之一标注：
- @Controller
- @Service
- @Repository
- @Component
在配置文件中配置**<context:annotation-config/>**元素来开启注解。

## 什么是依赖注入(DI)
DI是从另一个角度描述IoC。IoC是一种设计思想，DI是一种具体实现。想要实现控制反转最关键的就是将对象A所依赖的对象B，通过DI的方式给到对象A。
