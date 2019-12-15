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

## Spring中的核心模块
- Spring Core：核心类库，提供IoC服务；
- Spring Context：提供框架式的Bean访问方式，以及企业级功能(JNDI，定时任务等)；
- Spring AOP：
- Spring DAO：对JDBC进行了抽象，简化了数据访问异常等处理；
- Spring ORM：对现有的ORM持久层框架进行了支持；
- Spring Web：提供了基本的面向Web的综合特性；
- Spring MVC：提供面向Web应用的Model-View-Controller实现。

## Spring的优点有哪些？
- Spring的**依赖注入**将对象之间的依赖关系交给了框架来处理，减小了各个组件之间的耦合性。
- **AOP(面向切面编程)**，可以将通用的任务抽取出来，复用性更高。
- Spring对于其余主流框架都提供了很好的支持，代码侵入性很低。

## 什么是AOP？
面向切面编程。当需要向一个方法之前或之后进行一些额外的操作的时候，可以利用AOP将这些功能代码从业务代码中分离出来。例如：**日志记录**，**权限判断**，**异常统计**等等。
### AOP中涉及的术语
- Joinpoint(连接点)：类中能够被增强的方法。
- Pointcut(切入点)：实现了增强的方法。
- Advise(通知/增强)：用来“增强”其它方法的功能代码。
- Aspect(切面)：融合了切入点+增强的整体。
- Introduction(引介)：引介是一种特殊的通知在不修改类代码的前提下，它可以在运行期位类动态地添加一些方法或属性。
- Target(目标对象)：代理的目标对象(要增强的方法所在的类)。
- Weaving(织入)：将advise应用到target的过程。
- Proxy(代理)：一个类被AOP增强后，就会产生一个新的代理类，后续的针对该类的操作都是通过这个代理类实现的。

## Spring中AOP主要的实现方式：
- JDK动态代理，使用java.lang.reflection.Proxy类处理。
- 使用CGLib实现。
### 两者的区别
JDK动态代理只能对实现了接口的类生成代理，而不针对类，该目标类型实现的接口都被代理。原理是通过在运行期间创建一个接口的实现类来完成对目标对象的代理。
大概的实现步骤：
1. 定义一个实现接口的InvocationHandler的类；
2. 通过构造函数注入被代理类；
3. 实现invoke(Object proxy, Method method, Object[] args)方法；
4. 在主函数中获得被代理类的类加载器；
5. 使用Proxy.newProxyInstance()产生一个代理对象；
6. 通过代理对象调用各种方法。

CGLib主要针对类实现代理，对是否实现接口无要求。原理是对指定的类生成一个子类，覆盖其中的方法，因为是继承，所以被代理的类或方法不可以声明位final类型。
大致的实现步骤：
- 定义一个实现了MethodInterceptor接口的类；
- 实现该类的intercept()方法，该方法中调用proxy.invokeSuper()。

## IoC容器的初始化过程
- Resource资源定位：ResourceLoader通过同一的Resource接口来完成定位。
- BenDefinition的载入：把定义好的Bean表示成IoC容器内部的数据结构，即BeanDefinition。在配置文件中每一个Bean都对应着一个BeanDefinition对象。在IoC容器内部维护者一个BeanDefinitionMap的数据结构，通过BeanDefinitionMap，IoC容器可以对Bean进行更好的管理。
- BeanDefinition的注册：就将BeanDefinition保存到Map中的过程，通过BeanDefinitionRegistry接口来实现注册。

### BeanFactory和FactoryBean的区别
- BeanFactory：Bean工厂，是一个工厂(Factory)，是Spring IoC容器的顶层接口，它的作用是管理Bean，即**实例化，定位，配置**应用程序中的对象及建立这些对象间的依赖。
- FactoryBean：工厂Bean，是一个Bean，作用是产生其它Bean实例，提供一个工厂方法，该方法用来返回其他Bean实例。

### BeanFactory和ApplicationContext有什么区别？
- BeanFactory是Spring里面最顶层的接口，包含了各种Bean定义，读取Bean配置文档，管理Bean加载，实例化，控制Bean的生命周期，维护Bean之间的依赖关系。
- ApplicationContext接口时BeanFactory的派生，除了提供BeanFactory所具有的功能外，还提供了更完整的框架功能：
  ​	1. 继承了MessageSource，支持国际化。
  ​  2. 提供了统一的资源文件访问方式。
  ​  3. 提供在Listener中注册Bean的事件。
  ​  4. 提供同时加载多个配置文件的功能。
  ​  5. 载入多个(有继承关系)上下文，使得每一个上下文都专注于一个特定的层次。

### ApplicationContext三种常见的实现方式
- FileSystemXmlApplicationContext：从XML文件中加载Bean的定义，XML Bean配置文件的全路径名必须提供给它的构造函数。
- ClassPathXmlApplicationContext： 从XML文件中加载Bean的定义，需要正确设置classpath，因为该容器会从classpath里找Bean配置。
- WebXmlApplicationContext：加载一个XML文件，定义了一个Web应用的所有Bean。

### 在创建Bean和内存占用方面的区别：
- BeanFactory采用的是**延迟加载**的形式来注入Bean的，所有只有在第一次调用getBean方法的会发现Bean存在错误。
- ApplicationContext是在容器启动后，**一次性创建了所有的Bean**。这样在容器启动的时候就能够马上发现Spring中存在的配置错误。
- ApplicationContext更加消耗内存空间，并且在Bean比较多的时候，容器启动速度会比较慢。

### BeanFactory和ApplicationContenxt的优缺点
BeanFactory的优缺点
优点：应用启动的时候占用资源很少。
缺点：运行速度较慢；可能会出现空指针异常；创建的Bean生命周期会简单一些。

ApplicationContext的优缺点
优点：一次性加载了所有的Bean，程序运行的顺序快；在程序启动的时候，就可以发现程序中存在的配置问题。
缺点：占用内存大。

