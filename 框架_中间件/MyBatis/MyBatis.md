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

# #{}和${}的区别
**${}**是Properties文件中的变量占位符，它可以用于XML标签属性值和SQL内部，属于**字符串替换**。它也可以对传递进来的参数**原样拼接**在SQL中。

**#{}**是SQL的参数占位符，MyBatis会将SQL中的**#{}**替换为**?**，在SQL执行前会使用PreparedStatement的参数设置方法，按序给SQL的**?**占位符设置参数值。**#{}**是**预编译处理**，可以有效防止SQL注入，提高系统安全性。

# 如何解决实体类的属性名与表中的字段名不一样？
## 设置别名
在查询的SQL语句中定义字段名的别名，使别名与实体类的属性名一致。
## 开启自动转换
MyBatis支持将字段名的下划线命名法转为属性民的驼峰式命名法，需要在手动配置该特性。
```xml
<setting name="logImpl" value="LOG4J"/>
    <setting name="mapUnderscoreToCamelCase" value="true" />
</settings>
```
## 使用< resultMap >来映射字段名和属性名
```xml
<resultMap type="me.gacl.domain.Order" id=”OrderResultMap”> 
    <!–- 用 id 属性来映射主键字段 -–> 
    <id property="id" column="order_id"> 
    <!–- 用 result 属性来映射非主键字段，property 为实体类属性名，column 为数据表中的属性 -–> 
    <result property="orderNo" column ="order_no" /> 
    <result property="price" column="order_price" /> 
</resultMap>

<select id="getOrder" parameterType="Integer" resultMap="OrderResultMap">
    SELECT * 
    FROM orders 
    WHERE order_id = #{id}
</select>
```

# 动态SQL
MyBatis的这个特性能够让开发者在XML映射文件内，以XML标签的形式编写动态SQL，完成逻辑判断和动态拼接SQL的功能。主要是使用**OGNL表达式**，从SQL参数对象中计算表达式的值，根据表达式的值动态拼接SQL，从而实现动态SQL的功能。

## 动态SQL的标签
1. if
2. choose
3. when
4. otherwise
5. trim
6. where
7. set
8. forearch
9. bind

# Mapper接口
## 对应关系
1. 接口的全限名，就是映射文件中的**namespace**的值。
2. 接口的方法名，就是映射文件中MapperStatement的**id**值。
3. 接口方法内的参数，就是传递给SQL的参数。

## 绑定的实现方式
1. 在XML Mapper里写SQL来绑定，XML映射文件里面的**namespace**必须为接口的全路径名。
2. 通过**注解**绑定。
3. 通过**Provider注解**绑定。

## 传递多个参数
1. 使用Map(不推荐)。
2. 使用@Param注解。

# 执行器
1. SimpleExecutor：每执行一次update或select操作，就创建一个Statement对象，用完立刻关闭Statement对象。
2. ReuseExecutor：执行update或select操作，以SQL作为Key查找**缓存**的Statement对象，存在就使用，不存在就创建；用完后，不关闭Statement对象，而是放置于缓存Map中，供下一次使用。
3. BatchExecutor：执行update操作，将所有SQL都添加到批处理中，等待统一执行。
4. CachingExecutor：在上述的三个执行器之上，增加**二级缓存**的功能。


