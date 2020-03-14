# XML配置
[toc]

# 属性(properties)
这些属性可以在外部进行配置，并可以进行动态替换。

## 属性的配置
既可以在典型的Java属性文件中配置这些属性，也可以在properties元素的子元素中设置。
例如：
```xml
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>
```
设置好的属性可以在整个配置文件中用来替换需要动态配置的属性值。
例如：
```xml
<dataSource type="POOLED">
  <property name="driver" value="${driver}"/>
  <property name="url" value="${url}"/>
  <property name="username" value="${username}"/>
  <property name="password" value="${password}"/>
</dataSource>
```
也可以在SqlSessionFactoryBuilder.build()方法中传入属性值。
例如：
```java
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, props);
// ... 或者 ...
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment, props);
```
## 属性的加载顺序
**方法传递的参数的优先级最高，resource/url属性中指定的配置文件次之，优先级最低的是properties元素中指定的属性。**
如果一个属性在不只一个地方进行了配置，那么MyBatis将按照下面的顺序来加载：
1. 首先读取在properties元素体内指定的属性。
2. 然后根据properties元素中的resource属性读取类路径下属性文件，或根据url属性指定的路径读取属性文件，并覆盖之前读取过的同名属性。
3. 最后读取作为方法参数传递的属性，并覆盖之前读取过的同名属性。

## 占位符的默认值
从MyBatis 3.4.2开始，可以为占位符指定一个默认值。
例如：
```xml
<dataSource type="POOLED">
  <!-- ... -->
  <property name="username" value="${username:ut_user}"/> <!-- 如果属性 'username' 没有被配置，'username' 属性的值将为 'ut_user' -->
</dataSource>
```
该特性需要手动开启。
```xml
<properties resource="org/mybatis/example/config.properties">
  <!-- ... -->
  <property name="org.apache.ibatis.parsing.PropertyParser.enable-default-value" value="true"/> <!-- 启用默认值特性 -->
</properties>
```

## 特殊字符
如果在属性名中使用了**:**字符(如：**db:name**)，或者在SQL映射中使用了OGNL表达式的三元运算符，就需要设置特定的属性来修改分隔符属性名和默认值的字符。
例如：
```xml
<properties resource="org/mybatis/example/config.properties">
  <!-- ... -->
  <property name="org.apache.ibatis.parsing.PropertyParser.default-value-separator" value="?:"/> <!-- 修改默认值的分隔符 -->
</properties>
```
```xml
<dataSource type="POOLED">
  <!-- ... -->
  <property name="username" value="${db:username?:ut_user}"/>
</dataSource>
```

# 设置(settings)
这些设置会改变MyBatis的运行时行为。
[设置-官方文档](https://mybatis.org/mybatis-3/zh/configuration.html#settings)

# 类型别名(typeAliases)
类型别名能够为Java类型设置一个缩写名字。它仅用于XML配置，意在降低冗余的全限定类名书写。
例如：
```xml
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author"/>
  <typeAlias alias="Blog" type="domain.blog.Blog"/>
  <typeAlias alias="Comment" type="domain.blog.Comment"/>
  <typeAlias alias="Post" type="domain.blog.Post"/>
  <typeAlias alias="Section" type="domain.blog.Section"/>
  <typeAlias alias="Tag" type="domain.blog.Tag"/>
</typeAliases>
```
也可以指定一个包名。
例如：
```xml
<typeAliases>
  <package name="domain.blog"/>
</typeAliases>
```
使用注解给Java Bean设置别名。
例如：
```java
@Alias("author")
public class Author {
    ...
}
```

# 类型处理器
[类型处理器-官方文档](https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers)
MyBatis在设置预处理语句(PreparedStatement)中的参数或从结果集中取出一个值时，都会用类型处理器将获取到的值以合适的方式转换成Java类型。

通过实现**org.apache.ibatis.type.TypeHandler接口**或继承**org.apache.ibatis.type.BaseTypeHandler类**能够实现自定义的类型处理器。

# 对象工厂(objectFactory)
每次MyBatis创建结果对象的新实例时，它都会使用一个对象工厂实例来完成实例化工作。默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认无参构造方法，要么通过存在的参数映射来调用带有参数的构造方法。

# 环境配置(environments)
MyBatis可以配置成适应多种环境，这种机制有助于将SQL映射应用于多种数据库之中。

一个数据库需要一个SqlSessionFactory实例。


