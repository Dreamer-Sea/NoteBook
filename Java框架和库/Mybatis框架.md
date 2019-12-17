# MyBatis框架
[toc]
当下热门的Java持久层框架。一个**半ORM(对象关系映射)框架**，内部封装了JDBC，开发时只需要关注SQL语句本身，**支持动态SQL语句**。

## MyBatis与Hibernate的区别
- MyBatis的代码量少，容易上手，SQL语句和代码分离，数据库可移植。但是SQL语句需要自己写，并且参数只能有一个。
- Hibernate进行了对象关系数据库映射，完全面向对象，提供缓存机制和HQL编程。但是不能灵活使用原生SQL，无法对SQL优化，全表映射效率低下且存在N+1的问题。

## JDBC，MyBatis和Hibernate的主要区别
- JDBC更为灵活，更加效率，系统运行速度快。但是代码复杂，如果使用了存储过程就不方便数据库移植了。
- Hibernate和MyBatis是关系数据库框架，开发速度快，更加面对对象，可以移植更换数据库，但是影响系统性能。

## MyBatis有哪些核心组件
- SqlSessionFactoryBuilder；
- SqlSessionFactory；
- SqlSession；
- Mapper。
### SqlSessionFactoryBuilder
这是一个构造器，通过XML配置文件或者Java编码获得资源来构建SqlSessionFactory。通过Builder可以构建多个SessionFactory。其生命周期一般只存在于方法的局部，用完即可挥手。
### SqlSessionFactory
这个的作用就是创建SqlSession。每次程序需要访问数据库就会需要用到SqlSession。它应该在MyBatis应用的整个生命周期中。为了减少每次都创建一个会话带来的资源消耗，一般情况下都会使用单例模式来创建SqlSession。
### SqlSession
这是一个会话，相当于JDBC中的Connection对象，既可以发送SQL去执行并返回结果，也可以获取Mapper接口。这是个线程不安全的对象，它的生命周期应该是请求数据库处理事务的过程中。每次创建的SqlSession对象必须及时关闭，否则会使得数据库连接池的活动资源减少，影响系统性能。
### Mapper
映射器，由Java接口和XML文件共同组成，给出了对应的SQL和映射规则，主要负责发送SQL去执行，并且返回结果。

## 动态SQL
在XML映射文件中，以标签的形式编写动态SQL。实际上就是根据表达式的值完成逻辑判断并动态拼接SQL语句。
### 9种动态SQL标签：
- if：单条件分支的判断语句；
- choose，when，otherwise：多条件的分支判断语句；
- foreach：列举条件，遍历集合，实现循环语句；
- trim，where，set：是一些辅助元素，可以对拼接的SQL进行处理；
- bind：进行模糊判断查询的时候使用，提高数据库的可移植性。

## Mapper种常见的标签
- select | insert | update | delete
- resultMap | parameterMap | sql | include | selectKey
- trim | where | set | foreach | if | choose | when | otherwise | bind

## Dao接口的工作原理有了解吗？
Dao接口即Mapper接口。工作原理：使用**JDK动态代理**为Mapper接口生成代理对象proxy，代理对象会拦截接口方法，转而执行MapperStatement所代表的SQL，然后将SQL执行结果返回。
### Dao接口中的方法不可以重载
因为是使用**类的全名+方法名**的保持和寻找策略。
### 不同映射文件xml中的id值可以重复
- 不同的xml映射文件，如果配置了namespace，那么id可以重复。
- 如果没有配置namespace，那么id不能重复。

## MyBatis中\#和\$的区别
- \# 将传入的数据当作字符串，会对传入的数据加一对**双引号**。
- \$ 传入的数据**直接显示**在生成的SQL语句中。
- \# 符号**存在预编译的过程**，对问号赋值，防止**SQL注入**。
- \$ 符号是**直译**的方式，一般用在**order by  \$ {列名}**语句中。

## MyBatis的缓存机制
MyBatis的缓存机制分为一级和二级缓存。
- 一级缓存(同一个SqlSession)：
基于HashMap的本地缓存，其存储作用域为Session，当Session flush或closs之后，该Session中所有的缓存都将清空，默认打开一级缓存。
- 二级缓存(同一个SqlSessionFactory)：
二级缓存与一级缓存的机制仙童，默认也是采用HashMap的本地存储，不同在于其存储作用域为Mapper(Namespace)，并且可自定义存储园。

## MyBatis的接口绑定是什么？有哪些实现方式？
在MyBatis中任意的定义接口，然后把接口里的方法和SQL语句绑定，后续只需要直接调用接口方法就能执行对应SQL语句。
### 两种实现方式
- 注解：在接口方法上加上@Select，@Update等注解，里面包含具体的SQL语句从而实现绑定。
- XML文件：在XML中写SQL语句来绑定，namespace必须为接口的全路径名。
