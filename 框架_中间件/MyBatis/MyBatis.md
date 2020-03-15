# MyBatis

[toc]

# 优缺点
## 优点
1. 相比JDBC需要编写的代码更少。
2. 使用灵活，支持动态SQL。
3. 提供映射标签，支持对象与数据库的字段关系映射。
## 缺点
1. SQL语句依赖于数据库，数据库移植性差。
2. SQL语句编写工作量大，尤其是在表、字段比较多的情况下。

# 重要组件
1. Mapper配置：用于组织具体的查询业务和映射数据库的字段关系，可以使用XML格式或Java注解格式来实现。
2. Mapper接口：数据操作接口也就是通常说的DAO接口，要和Mapper配置文件中的方法一一对应。
3. Executor：MyBatis中所有的Mapper语句的执行都是通过Executor执行的。
4. SqlSession：类似于JDBC中的Connection，可以用SqlSession实例来直接执行被映射的SQL语句。
5. SqlSessionFactory：SqlSessionFactory是创建SqlSession的工厂，可以通过SqlSession.openSession()方法创建SqlSession对象。

# 执行流程
![MyBatis执行流程](/框架_中间件/MyBatis/pictures/MyBatis执行流程.jpg)

1. 首先加载Mapper配置的SQL映射文件，或者是注解的相关SQL内容。
2. 创建会话工厂，MyBatis通过读取配置文件的信息来构造出会话工厂(SqlSessionFactory)。
3. 创建会话，根据会话工厂，MyBatis就可以通过它来创建会话对象(SqlSession)，会话对象是一个接口，该接口中包含了对数据库操作的增、删、改、查方法。
4. 创建执行器：因为会话对象本身不能直接操作数据库，所以它使用了一个叫做数据库执行其(Executor)的接口来帮它执行操作。
5. 封装SQL对象，在这一步，执行器将待处理的SQL信息封装到一个对象中(MappedStatement)，该对象包括SQL语句、输入参数映射信息(Java简单类型、HashMap或POJO)和输出接口映射信息(Java简单类型、HashMap或POJO)。
6. 操作数据库，拥有了执行器和SQL信息封装对象就使用它们访问数据库了，最后再返回操作结果，结束流程。