

# Java框架

## Mybatis

![MybatisLogo](E:\files\JAVA\笔记\2021\resources\mybatis\MybatisLogo.png)

### 1.简介

#### 什么是Mybatis

- MyBatis是一款优秀的**持久型框架**
- 它支持定制化SQL、存储过程以及高级映射.
- MyBatis避免了几乎所有的JDBC代码和手动设置参数以及获取结果集.
- Mybatis可以使用简单的XML或注解来配置和映射原生类型、接口和Java的POJO(Plain Old Java Objects,普通老式Java对象)为数据库中的记录



如何获得MyBatis

- Maven仓库

  ```xml
  <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
  <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.2</version>
  </dependency>
  
  ```

  

- Github:https://github.com/mybatis/mybatis-3/releases

- 中文文档:https://mybatis.org/mybatis-3/zh/index.html



#### 持久化

数据持久化:

- 持久化就是将程序的数据在持久状态和瞬时状态转化的过程
- 内存:**断电即失**
- 数据库(jdbc)、io文件持久化



#### **为什么需要持久化?**

有一些对象,不能丢掉



#### 持久层

Dao层,Service层,Controller层...

- 完成持久化工作的代码块
- 层界线十分明显



#### 为什么需要Mybatis

- 方便
- 传统的JDBC代码太复杂.简化.框架.自动化
- 帮助程序员将数据存入到数据库中
- 不用Mybatis也可以
- 优点:
  - 简单易学
  - 灵活
  - sql和代码分离,提高了可维护性
  - 提供映射标签,支持对象与数据库的orm字段关系映射
  - 提供对象关系映射标签,支持对象关系组件维护
  - 提供xml标签,支持编写动态sql



### 2.第一个MyBatis程序

思路: 搭建环境 - 导入MyBatis - 编写代码 - 测试



#### 搭建环境

搭建数据库

```mysql
CREATE DATABASE `mybatis`;

USE `mybatis`

CREATE TABLE `user`(
	`id` INT(20) PRIMARY KEY,
	`name` VARCHAR(30) DEFAULT NULL,
	`pwd` VARCHAR(30) DEFAULT NULL
)ENGINE=INNODB default charset=utf8;

INSERT INTO `user`(`id` , `name` , `pwd`) VALUES(1 , '你爷爷' , '123456'),(2 , '你爸' , '12312356'),(3 , '你妈' , '123412356');
```


新建项目

1. 创建一个普通maven项目
2. 删除src使其成为父工程
3. 导入maven依赖
   1. mysql-connector
   2. mybatis
   3. junit

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!--  父工程  -->
    <groupId>com.nidie</groupId>
    <artifactId>Mybatis-Study</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!--  导入依赖  -->
    <dependencies>
        <!--   mysql驱动   -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
        <!--    mybatis    -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.2</version>
        </dependency>
        <!--    junit    -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
    </dependencies>

</project>
```



#### 创建一个项目

- 编写Mybatis的核心配置文件

  ```java
  String resource = "org/mybatis/example/mybatis-config.xml";
  InputStream inputStream = Resources.getResourceAsStream(resource);
  SqlSessionFactory sqlSessionFactory = new SqlSessionFactory().build(inputStream);
  ```

  

- 编写mybatis工具类

```java
package com.nidie.utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

//sqlSessionFactory --> sqlSession
public class MybatisUtils {

    private static SqlSessionFactory sqlSessionFactory;

    static{
        try {
//          使用Mybatis第一步、获取sqlSessionFactory对象
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /*
    *  既然有了SqlSessionFactory,顾名思义,我们就可以从中获取SqlSession的实例了.SqlSession完全包含了
    *  面向数据库执行SQL命令所需的所有方法
    * */

    public static SqlSession getSqlSession(){
        return sqlSessionFactory.openSession();
    }

}

```



#### 编写代码

- 实体类

  ```java
  package com.nidie.pojo;
  
  public class User {
  
      private int id;
  
      private String name;
  
      private String pwd;
  
      public User() {
      }
  
      public User(int id, String name, String pwd) {
          this.id = id;
          this.name = name;
          this.pwd = pwd;
      }
  
      public int getId() {
          return id;
      }
  
      public void setId(int id) {
          this.id = id;
      }
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      public String getPwd() {
          return pwd;
      }
  
      public void setPwd(String pwd) {
          this.pwd = pwd;
      }
  
      @Override
      public String toString() {
          return "User{" +
                  "id=" + id +
                  ", name='" + name + '\'' +
                  ", pwd='" + pwd + '\'' +
                  '}';
      }
      
  }
  
  ```

  

- Dao接口

```java
public interface UserDao {

    List<User> getUserList();

}
```

- 接口实现类由原来的UserDaoImp转换为一个Mapper配置文件

```xml
<?xml version = "1.0" encoding = "UTF-8" ?>
<!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace=绑定一个对应的Mapper/Dao接口-->
<mapper namespace = "com.nidie.Dao.UserDao">

    <!--select查询语句-->
    <select id="getUserList" resultType="com.nidie.pojo.User">
        select * from mybatis.user
    </select>

</mapper>
```



#### 测试

注意点:

```java
org.apache.ibatis.binding.BindingException: Type interface com.nidie.Dao.UserDao is not known to the MapperRegistry.

	at org.apache.ibatis.binding.MapperRegistry.getMapper(MapperRegistry.java:47)
	at org.apache.ibatis.session.Configuration.getMapper(Configuration.java:779)
	at org.apache.ibatis.session.defaults.DefaultSqlSession.getMapper(DefaultSqlSession.java:291)
	at com.nidie.Dao.UserDaoTest.test(UserDaoTest.java:18)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:50)
	at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
	at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:47)
	at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
	at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:325)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:78)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:57)
	at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
	at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
	at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:68)
	at com.intellij.rt.execution.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:47)
	at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:242)
	at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:70)
```



MapperRegistry是什么?

核心配置文件中注册mappers



```java
java.lang.ExceptionInInitializerError
	at com.nidie.Dao.UserDaoTest.test(UserDaoTest.java:15)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:50)
	at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
	at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:47)
	at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
	at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:325)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:78)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:57)
	at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
	at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
	at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:68)
	at com.intellij.rt.execution.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:47)
	at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:242)
	at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:70)
Caused by: org.apache.ibatis.exceptions.PersistenceException: 
### Error building SqlSession.
### The error may exist in com/nidie/Dao/UserMapper.xml
### Cause: org.apache.ibatis.builder.BuilderException: Error parsing SQL Mapper Configuration. Cause: java.io.IOException: Could not find resource com/nidie/Dao/UserMapper.xml
	at org.apache.ibatis.exceptions.ExceptionFactory.wrapException(ExceptionFactory.java:30)
	at org.apache.ibatis.session.SqlSessionFactoryBuilder.build(SqlSessionFactoryBuilder.java:80)
	at org.apache.ibatis.session.SqlSessionFactoryBuilder.build(SqlSessionFactoryBuilder.java:64)
	at com.nidie.utils.MybatisUtils.<clinit>(MybatisUtils.java:21)
	... 23 more
Caused by: org.apache.ibatis.builder.BuilderException: Error parsing SQL Mapper Configuration. Cause: java.io.IOException: Could not find resource com/nidie/Dao/UserMapper.xml
	at org.apache.ibatis.builder.xml.XMLConfigBuilder.parseConfiguration(XMLConfigBuilder.java:121)
	at org.apache.ibatis.builder.xml.XMLConfigBuilder.parse(XMLConfigBuilder.java:98)
	at org.apache.ibatis.session.SqlSessionFactoryBuilder.build(SqlSessionFactoryBuilder.java:78)
	... 25 more
Caused by: java.io.IOException: Could not find resource com/nidie/Dao/UserMapper.xml
	at org.apache.ibatis.io.Resources.getResourceAsStream(Resources.java:114)
	at org.apache.ibatis.io.Resources.getResourceAsStream(Resources.java:100)
	at org.apache.ibatis.builder.xml.XMLConfigBuilder.mapperElement(XMLConfigBuilder.java:372)
	at org.apache.ibatis.builder.xml.XMLConfigBuilder.parseConfiguration(XMLConfigBuilder.java:119)
	... 27 more
```



- junit测试

```java
@Test
    public void test(){
//      第一步: 获得SqlSession对象
        SqlSession sqlSession = MybatisUtils.getSqlSession();

//      方式一: getMapper
        UserDao mapper = sqlSession.getMapper(UserDao.class);
        List<User> userList = mapper.getUserList();

        for (User u:
             userList) {
            System.out.println(u.toString());
        }

//      关闭SqlSession
        sqlSession.close();
    }
```





#### 防止资源导出失败的问题

```xml
<!--在build中配置resources,来防止我们资源到处失败的问题-->
    <build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
    </resources>
</build>
```



- 可能会遇到的问题:
  1. 配置文件没注册
  2. 绑定接口错误
  3. 方法名不对
  4. 返回类型不对
  5. Maven导出资源问题



### 3.CRUD

#### namespace

namespace中的包名要和Dao/mapper中的包名一致



#### select

选择,查询语句:

- id:就是对应的namespace中的方法名
- resultType: Sql语句执行的返回值
- parameterType:参数类型



#### insert

增删改要提交事务 sqlSession.commit();



1.编写接口

```java
int addUser(User user);
```

2.编写对应的mapper中的sql语句

```xml
<insert id="addUser" parameterType="com.nidie.pojo.User">
        insert into mybatis.user (name , pwd) values (#{name} , #{pwd})
</insert>
```

3.测试

```java
@Test
    public void addUser(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        int result = mapper.addUser(new User("你大爸" , "4615112365"));

        System.out.println(result);

        //提交事务
        sqlSession.commit();
        sqlSession.close();
    }
```



#### update



#### delete



#### 分析错误

1. 便签不匹配
2. resource绑定mapper,需要使用路径
3. 程序配置文件必须符合规范
4. NullPointerException,没有注册到资源
5. 输出的xml文件中存在中文乱码问题
6. maven资源没导出问题



#### 万能Map

假设,我们的实体类,或者数据库中的表,字段或者参数过多,我们应当考虑使用Map

```java
//    万能的Map
    int addUser2(Map<String , Object> map);
```

```xml
<!--对象中的属性,可以直接取出来,传递map的key-->
    <insert id="addUser2" parameterType="map">
        insert into mybatis.user(name, pwd) values(#{username} , #{password})
    </insert>
```

```java
@Test
    public void addUser2(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        Map<String, Object> map = new HashMap<String, Object>();

        map.put("username" , "确实帅");
        map.put("password" , "queshishuai");

        mapper.addUser2(map);

        //提交事务
        sqlSession.commit();
        sqlSession.close();
    }
```

Map传递参数,直接在sql中取出key即可		parameterType = "map"

对象传递参数,直接在sql中取对象的属性即可		parameterType = "Object"

只有一个基本类型参数的情况下,可以直接在sql中取到



多个参数用Map,**或者注解**



#### 思考题

模糊查询怎么写?

1.java代码执行的时候,传递通配

```java
List<User> users = mapper.getUserLike("%帅%");
```

2.在sql拼接中使用通配符

```xml
select * from mybatis.user where name like "%"#{value}"%"
```



### 4.配置解析

#### 核心配置文件

- mybatis-config.xml
- MyBatis的配置文件包含了深深影响MyBatis行为的设置和属性信息机.配置文档的顶层结构如下:
  - configuration(配置)
    - properties(属性)
    - settings(设置)
    - typeAliases(类型别名)
    - typeHandlers(类型处理器)
    - objectFactory(对象工厂)
    - plugins(插件)
    - environments(环境配置)
      - environment(环境变量)
        - transactionManager(事务管理器)
        - dataSource(数据源)
    - databseIdProvider(数据库厂商标识)
    - mappers(映射器)



##### 环境配置(environments)

MyBatis可以配置成适应多种环境

**不过要记住:尽管可以配置多个环境,但每个SqlSessionFactory实例只能选择一种环境**

环境元素定义了如何配置环境

```xml
	<environments default="test">
        <environment id="development">
            <transactionManager type="JDBC" />
            <dataSource type="POOLED">
                <property name = "driver" value = "com.mysql.jdbc.Driver"/>
                <property name = "url" value = "jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8" />
                <property name = "username" value = "root" />
                <property name = "password" value = "" />
            </dataSource>
        </environment>
        
        <environment id="test">
            <transactionManager type="JDBC" />
            <dataSource type="POOLED">
                <property name = "driver" value = "com.mysql.jdbc.Driver"/>
                <property name = "url" value = "jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8" />
                <property name = "username" value = "root" />
                <property name = "password" value = "" />
            </dataSource>
        </environment>
        
    </environments>
```

这里的关键点:

- 默认使用的环境ID(比如: default = "development")
- 每个environment元素定义的环境ID (比如: id = "development")
- 事务管理器的配置(比如: type = "JDBC")
- 数据源的配置(比如: type = "POOLED")

默认的环境和环境ID是自解释的,因此一目了然.你可以对环境随意命名,但一定要保证默认的环境ID要匹配其中一个环境ID



- dbcp
- c3p0
- druid



Mybatis默认的事务管理器是JDBC,默认连接池:POOLED



##### 属性(properties)

我们可以通过properties属性来实现引用配置文件

这些属性都是可外部配置且可动态替换的,既可以在典型的Java属性文件中配置,亦可通过properties元素的的子元素来传递

编写一个配置文件

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8
username=root
password=
```

在核心配置文件中映入

```xml
<!--    <properties>-->
<!--        <property name="xxx" value="xxxx"/>-->
<!--        <property name="xxx" value="xxxx"/>-->
<!--    </properties>-->
```



- 可以直接引入外部文件
- 可以在其中增加一些属性配置
- 



mybatis-config.xml文件的标签顺序

1. properties
2. settings
3. typeAlias
4. typeHandlers
5. objectFactory
6. objectWrapperFactory
7. reflectorFactory
8. plugins
9. environments
10. databaseIdProvider
11. mappers





##### 类别别名(TypeAlias)

- 类型别名是为Java类型设置一个短的名字
- 存在的意义仅在于用来减少类完全限定名的冗余

```xml
<!-- 方式1,直接给类取别名 -->
<typeAlias type="com.nidie.pojo.User" alias="User" />
```

也可以指定一个包名,MyBatis会在包名下面搜索需要的Java Bean,比如:

扫描实体类的包名,它的默认别名就为这个类的类名,首字母小写

```xml
<package name="com.nidie.pojo"/>
```



在实体类比较小的时候,使用第一种方式

如果实体类较多,建议使用第二种

第一种可以自定义别名,第二种不行



扫描包的情况下可以通过注解取别名

```java
@Alias("hello")
public class User {
    xxx
}
```





##### 设置(settings)

这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。 下表描述了设置中各项设置的含义、默认值等。

| 设置名                           | 描述                                                         | 有效值                                                       | 默认值                                                |
| :------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :---------------------------------------------------- |
| **cacheEnabled**                 | 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。     | true \| false                                                | true                                                  |
| **lazyLoadingEnabled**           | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态。 | true \| false                                                | false                                                 |
| aggressiveLazyLoading            | 开启时，任一方法的调用都会加载该对象的所有延迟加载属性。 否则，每个延迟加载属性会按需加载（参考 `lazyLoadTriggerMethods`)。 | true \| false                                                | false （在 3.4.1 及之前的版本中默认为 true）          |
| multipleResultSetsEnabled        | 是否允许单个语句返回多结果集（需要数据库驱动支持）。         | true \| false                                                | true                                                  |
| useColumnLabel                   | 使用列标签代替列名。实际表现依赖于数据库驱动，具体可参考数据库驱动的相关文档，或通过对比测试来观察。 | true \| false                                                | true                                                  |
| useGeneratedKeys                 | 允许 JDBC 支持自动生成主键，需要数据库驱动支持。如果设置为 true，将强制使用自动生成主键。尽管一些数据库驱动不支持此特性，但仍可正常工作（如 Derby）。 | true \| false                                                | False                                                 |
| autoMappingBehavior              | 指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示关闭自动映射；PARTIAL 只会自动映射没有定义嵌套结果映射的字段。 FULL 会自动映射任何复杂的结果集（无论是否嵌套）。 | NONE, PARTIAL, FULL                                          | PARTIAL                                               |
| autoMappingUnknownColumnBehavior | 指定发现自动映射目标未知列（或未知属性类型）的行为。`NONE`: 不做任何反应`WARNING`: 输出警告日志（`'org.apache.ibatis.session.AutoMappingUnknownColumnBehavior'` 的日志等级必须设置为 `WARN`）`FAILING`: 映射失败 (抛出 `SqlSessionException`) | NONE, WARNING, FAILING                                       | NONE                                                  |
| defaultExecutorType              | 配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（PreparedStatement）； BATCH 执行器不仅重用语句还会执行批量更新。 | SIMPLE REUSE BATCH                                           | SIMPLE                                                |
| defaultStatementTimeout          | 设置超时时间，它决定数据库驱动等待数据库响应的秒数。         | 任意正整数                                                   | 未设置 (null)                                         |
| defaultFetchSize                 | 为驱动的结果集获取数量（fetchSize）设置一个建议值。此参数只可以在查询设置中被覆盖。 | 任意正整数                                                   | 未设置 (null)                                         |
| defaultResultSetType             | 指定语句默认的滚动策略。（新增于 3.5.2）                     | FORWARD_ONLY \| SCROLL_SENSITIVE \| SCROLL_INSENSITIVE \| DEFAULT（等同于未设置） | 未设置 (null)                                         |
| safeRowBoundsEnabled             | 是否允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为 false。 | true \| false                                                | False                                                 |
| safeResultHandlerEnabled         | 是否允许在嵌套语句中使用结果处理器（ResultHandler）。如果允许使用则设置为 false。 | true \| false                                                | True                                                  |
| **mapUnderscoreToCamelCase**     | 是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。 | true \| false                                                | False                                                 |
| localCacheScope                  | MyBatis 利用本地缓存机制（Local Cache）防止循环引用和加速重复的嵌套查询。 默认值为 SESSION，会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地缓存将仅用于执行语句，对相同 SqlSession 的不同查询将不会进行缓存。 | SESSION \| STATEMENT                                         | SESSION                                               |
| jdbcTypeForNull                  | 当没有为参数指定特定的 JDBC 类型时，空值的默认 JDBC 类型。 某些数据库驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。 | JdbcType 常量，常用值：NULL、VARCHAR 或 OTHER。              | OTHER                                                 |
| lazyLoadTriggerMethods           | 指定对象的哪些方法触发一次延迟加载。                         | 用逗号分隔的方法列表。                                       | equals,clone,hashCode,toString                        |
| defaultScriptingLanguage         | 指定动态 SQL 生成使用的默认脚本语言。                        | 一个类型别名或全限定类名。                                   | org.apache.ibatis.scripting.xmltags.XMLLanguageDriver |
| defaultEnumTypeHandler           | 指定 Enum 使用的默认 `TypeHandler` 。（新增于 3.4.5）        | 一个类型别名或全限定类名。                                   | org.apache.ibatis.type.EnumTypeHandler                |
| callSettersOnNulls               | 指定当结果集中值为 null 的时候是否调用映射对象的 setter（map 对象时为 put）方法，这在依赖于 Map.keySet() 或 null 值进行初始化时比较有用。注意基本类型（int、boolean 等）是不能设置成 null 的。 | true \| false                                                | false                                                 |
| returnInstanceForEmptyRow        | 当返回行的所有列都是空时，MyBatis默认返回 `null`。 当开启这个设置时，MyBatis会返回一个空实例。 请注意，它也适用于嵌套的结果集（如集合或关联）。（新增于 3.4.2） | true \| false                                                | false                                                 |
| logPrefix                        | 指定 MyBatis 增加到日志名称的前缀。                          | 任何字符串                                                   | 未设置                                                |
| **logImpl**日志实现              | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。        | SLF4J \| LOG4J \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | 未设置                                                |
| proxyFactory                     | 指定 Mybatis 创建可延迟加载对象所用到的代理工具。            | CGLIB \| JAVASSIST                                           | JAVASSIST （MyBatis 3.3 以上）                        |
| vfsImpl                          | 指定 VFS 的实现                                              | 自定义 VFS 的实现的类全限定名，以逗号分隔。                  | 未设置                                                |
| useActualParamName               | 允许使用方法签名中的名称作为语句参数名称。 为了使用该特性，你的项目必须采用 Java 8 编译，并且加上 `-parameters` 选项。（新增于 3.4.1） | true \| false                                                | true                                                  |
| configurationFactory             | 指定一个提供 `Configuration` 实例的类。 这个被返回的 Configuration 实例用来加载被反序列化对象的延迟加载属性值。 这个类必须包含一个签名为`static Configuration getConfiguration()` 的方法。（新增于 3.2.3） | 一个类型别名或完全限定类名。                                 | 未设置                                                |
| shrinkWhitespacesInSql           | 从SQL中删除多余的空格字符。请注意，这也会影响SQL中的文字字符串。 (新增于 3.5.5) | true \| false                                                | false                                                 |
| defaultSqlProviderType           | Specifies an sql provider class that holds provider method (Since 3.5.6). This class apply to the `type`(or `value`) attribute on sql provider annotation(e.g. `@SelectProvider`), when these attribute was omitted. | A type alias or fully qualified class name                   | Not set                                               |



##### 其他配置

- [typeHandlers（类型处理器）](https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers)
- [objectFactory（对象工厂）](https://mybatis.org/mybatis-3/zh/configuration.html#objectFactory)
- [plugins（插件）](https://mybatis.org/mybatis-3/zh/configuration.html#plugins)
  - MyBatis Generator Core自动编写MyBatis代码
  - MyBatis Plus第三方扩展
  - 通用mapper



##### 映射器(mappers)

MapperRegistry:注册绑定我们的mapper文件;

四种映射方式:

1.使用相对于类路径的资源引用

```xml
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>
```

```xml
<mappers>
        <mapper resource="com/nidie/dao/UserMapper.xml" />
</mappers>
```

2.使用完全限定资源定位符(URL):不推荐使用

```xml
<!-- 使用完全限定资源定位符（URL） -->
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <mapper url="file:///var/mappers/BlogMapper.xml"/>
  <mapper url="file:///var/mappers/PostMapper.xml"/>
</mappers>
```

3.使用映射器接口实现类的完全限定类名

```xml
<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>
```

使用class绑定

```xml
<mapper class="com.nidie.dao.UserMapper" />
```

注意点:

- 接口和他的Mapper配置文件必须同名
- 接口和他的Mapper配置文件必须在同一个包下

4.将包内的映射器接口实现全部注册为映射器

```xml
<!-- 将包内的映射器接口实现全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```

```xml
<package name="com.nidie.dao"/>
```

注意点:

- 接口和他的Mapper配置文件必须同名
- 接口和他的Mapper配置文件必须在同一个包下



练习:

1. 将数据库配置文件外部引入
2. 实体类别名
3. 保证UserMapper接口和UserMapper.xml改为一致!并且放在同一个包下



##### 生命周期和作用域

![mybatis1](E:\files\JAVA\笔记\2021\resources\mybatis\mybatis1.png)

不同作用域和生命周期类别是至关重要的，因为错误的使用会导致非常严重的**并发问题**。



SqlSessionFactoryBuilder:

- 一旦创建了SqlSessionFactoryBuilder,就不用了
- 局部变量

SqlSessionFactory:

- 可以想象为:数据库连接池
-  一旦被创建就应该在应用的运行期间一直存在，**没有任何理由丢弃它或重新创建另一个实例**。
- 因此SqlSessionFactory的最佳作用域就是应用作用域
- 最简单的就是使用**单例模式**或者静态单例模式

SqlSession:

- 连接到连接池的一个请求
- 请求完需要关闭
- SqlSession 的实例不是线程安全的，因此是不能被共享的
- 它的最佳的作用域是请求或方法作用域
- 用完之后需要赶紧关闭,否则资源被占用

![mybatis2](E:\files\JAVA\笔记\2021\resources\mybatis\mybatis2.png)

这里面的每一个mapper代表具体的业务



### 5.解决属性名和字段名不一致的问题

#### 问题

数据库中的字段

![mybatis3](E:\files\JAVA\笔记\2021\resources\mybatis\mybatis3.png)

新建一个项目,拷贝之前的,测试实体类字段不一致的情况

```java
public void User{
    private int id;
    
    private String name;
    
    private String password;
}
```

![mybatis5](E:\files\JAVA\笔记\2021\resources\mybatis\mybatis5.png)

```xml
select * from mybatis.user where id = #{id}
select id,name,pwd from mybatis.user where id = #{id}
```

解决方案:

- 起别名

  ```xml
  <select id="getUserById" resultType="com.nidie.pojo.User">
          select id, name, pwd AS password from mybatis.user where id = #{id}
      </select>
  ```



#### ResultMap

结果集映射

```xml
id name pwd
id name password
```

```xml
<resultMap id="UserMap" type="User">
        <!--colum数据库中的字段,property实体类中的属性-->
        <result column="id" property="id" />
        <result column="name" property="name" />
        <result column="pwd" property="password" />
    </resultMap>

    <select id="getUserById" resultType="User" resultMap="UserMap">
        select * from mybatis.user where id = #{id}
    </select>
```



- ResultMap余安安素是MyBatis中最重要最强大的元素
- ResultMap的设计思想是,对于简单的语句根本不需要配置显式的结果映射,而对于复杂一点的语句只需要描述他们的关系就行了
- ResultMap最优秀的地方在于,虽然你已经对它相当了解了,但是根本就不需要显式地用到它们







### 6.日志

#### 日志工厂

如果一个数据库操作,出现了异常,我们需要排错.日志就是最好的助手

曾经:sout、debug

现在:日志工厂

![mybatis4](E:\files\JAVA\笔记\2021\resources\mybatis\mybatis4.png)

- SLF4J 
- LOG4J [掌握]
- LOG4J2 
- JDK_LOGGING 
- COMMONS_LOGGING 
- STDOUT_LOGGING [掌握]
- NO_LOGGING

在Mybatis中具体使用哪一个日志实现,在设置中设定



##### STDOUT_LOGGING**标准输出**

在mybatis核心文件中,配置日志

```xml
	<settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
```

![mybatis6](E:\files\JAVA\笔记\2021\resources\mybatis\mybatis6.png)

Opening JDBC Connection 正在打开JDBC链接
Created connection 614685048. 创建连接
Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@24a35978] 在JDBC连接中将自动提交设为false
==>  Preparing: select * from mybatis.user where id = ?  准备SQL语句
==> Parameters: 1(Integer)	参数
<==    Columns: id, name, pwd	列
<==        Row: 1, 你爷爷, 123456	行
<==      Total: 1	总计
User{id=1, name='你爷爷', password='123456'}	输出结果
Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@24a35978]	重新将JDBC连接中的自动提交设为true
Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@24a35978]	关闭JDBC连接中
Returned connection 614685048 to pool.	将连接返回连接池



##### LOG4J

Log4j是什么

- Log4j是Apache的一个开源项目,通过使用Log4j,我们可以控制日志信息输送的目的地是控制台、文件、GUI组件
- 我们可以控制每一条日志的输出格式
- 通过定义每一条日志信息的级别,我们能够更加细致地控制日志的生成过程
- 通过一个配置文件来灵活地进行配置,而不需要修改应用的代码



1. 先导包

   ```xml
   <!-- https://mvnrepository.com/artifact/log4j/log4j -->
   <dependency>
       <groupId>log4j</groupId>
       <artifactId>log4j</artifactId>
       <version>1.2.17</version>
   </dependency>
   ```

2. log4j.properties

   ```xml
   #将等级为DEBUG的日志信息输出到console和file这两个目的地,console和file的定义在下面代码
   log4j.rootLogger=DEBUG,console,file
   
   #控制台输出的相关设置
   log4j.appender.console=org.apache.log4j.ConsoleAppender
   log4j.appender.console.Target=System.out
   log4j.appender.console.Threshold=DEBUG
   log4j.appender.console.layout=org.apache.log4j.PatternLayout
   log4j.appender.console.layout.ConversionPattern=[%c]-%m%n
   
   #文件输出的相关设置
   log4j.appender.file=org.apache.log4j.RollingFileAppender
   log4j.appender.file.File=./log/nidie.log
   log4j.appender.file.MaxFileSize=10mb
   log4j.appender.file.Threshold=DEBUG
   log4j.appender.file.layout=org.apache.log4j.PatternLayout
   log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n
   
   #日志输出级别
   log4j.logger.org.mybatis=DEBUG
   log4j.logger.java.sql=DEBUG
   log4j.logger.java.sql.Statement=DEBUG
   log4j.logger.java.sql.ResultSet=DEBUG
   log4j.logger.java.sql.PreparedStatement=DEBUG
   ```

   

3. 配置log4J为日志的实现

   ```
   <settings>
   	<setting name="logImpl" value="" />
   </settings>
   ```

4. Log4J的使用,直接测试运行刚才的查询


![mybatis7](E:\files\JAVA\笔记\2021\resources\mybatis\mybatis7.png)

**简单使用**

1. 在要使用Log4J的类中导入包

   ```java
   import org.apache.log4j.Logger;
   ```

2. 生成日志对象,参数为当前的class

   ```java
   static Logger logger = Logger.getLogger(UserMapperTest.class);
   ```

3. 日志级别

   ```java
   	logger.info("我是你爹info:进入了TestLog4j");
   	logger.debug("我是你爹debug:进入了testLog4j");
   	logger.error("我是你爹error:进入了testLog4j");
   ```





### 7.分页

**思考:为什么要分页?**

- 减少数据的处理量



#### **使用Limit分页**

```sql
语法:SELECT * FROM user LIMIT startIndex,pageSize;
SELECT * FROM user LIMIT 3; #[0,n]
```

使用Mybatis实现分页，核心SQL

- 接口

  ```java
  List<User> getUserByLimit(Map<String , Integer> map);
  ```

  

- Mapper.xml

  ```xml
      <select id="getUserByLimit" parameterType="map" resultMap="UserMap">
          select * from mybatis.user limit #{startIndex},#{pageSize}
      </select>
  ```

  

- 测试

```java
    @Test
    public void getUserByLimit(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        Map<String , Integer> map = new HashMap<String, Integer>();
        map.put("startIndex" , 1);
        map.put("pageSize" , 2);

        List<User> userLimit = mapper.getUserByLimit(map);

        for (User user:
             userLimit) {
            System.out.println(user);
        }

        sqlSession.close();
    }
```





#### RowBounds分页

不使用SQL实现分页

1. 接口

   ```java
   List<User> getUserByRowBounds();
   ```

2. mapper.xml

   ```xml
   <!--分页2-->
       <select id="getUserByRowBounds" parameterType="map" resultMap="UserMap">
           select * from mybatis.user
       </select>
   ```

3. 测试

```java
    @Test
    public void getUserByRowBounds(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        //RowBounds实现
        RowBounds rowBounds = new RowBounds(1 , 2);


        //通过Java代码层面实现分页
        List<User> userList = sqlSession.selectList("com.nidie.dao.UserMapper.getUserByRowBounds" , null , rowBounds);
        for (User user:
             userList) {
            System.out.println(user);
        }
        
        sqlSession.close();
    }

```





#### 分页插件

https://pagehelper.github.io/





### 8.使用注解开发

- 真正开发中,很多时候会选择面向接口编程
- 根本原因:**解耦**,提高复用,分层开发中,上层不用管具体的实现,大家都遵守共同的标准,使得开发变得容易,规范性更好
- 在一个面向对象的系统中,系统的各种功能是由许许多多的不同对象协作完成的.在这种情况下,各个对象内部是如何实现自己的,对系统设计人员来讲就不那么重要了
- 而各个对象之间的协作关系则成为系统设计的关键.小到不同类之间的通信,大到各模块之间的交互,在系统设计之初都是要着重考虑的,这也是系统设计的主要工作内容.面向接口编程就是指按照这种思路来编程



**关于接口的理解**

- 接口聪更深层次的理解,应是定义(规范,约束)与实现(名实分离的原则)的分离
- 接口的本身反映了系统设计人员对系统的抽象理解
- 接口应有两类:
  - 第一类是对一个个体的抽象,它可对应为一个抽象体(abstract class)
  - 第二类是对一个个体某一方面的抽象,即形成一个抽象面(interface)
- 一个体有可能有多个抽象面。抽象体和抽象面是有区别的



三个面向区别

- 面向对象是指,我们考虑问题时,以对象为单位,考虑它的属性及方法
- 面向过程是指,我们考虑问题是,以一个具体的流程(事务过程)为单位,考虑它的实现
- 接口设计与非接口设计是针对复用技术而言的,与面相对象(过程)不是一个问题.更多的体现就是对系统整体的架构



#### 使用注解开发

1. 注解在接口上实现

   ```java
       @Select("select * from mybatis.user")
       List<User> getUserList();
   ```

   

2. 需要在核心配置文件中绑定接口

   ```xml
       <mappers>
           <mapper class="com.nidie.dao.UserMapper" />
       </mappers>
   ```

   

3. 测试



本质:反射机制实现

底层:动态代理



![mybatis8](E:\files\JAVA\笔记\2021\resources\mybatis\mybatis8.png)

**Mybatis详细的执行流程**

![mybatis9](E:\files\JAVA\笔记\2021\resources\mybatis\mybatis9.png)



#### CRUD

我们可以在工具类创建的时候实现自动提交事务

```java
public SqlSession openSession() {
        return this.openSessionFromDataSource(this.configuration.getDefaultExecutorType(), (TransactionIsolationLevel)null, false);
    }
```

默认openSession()不自动提交事务

```xml
    public SqlSession openSession(boolean autoCommit) {
        return this.openSessionFromDataSource(this.configuration.getDefaultExecutorType(), (TransactionIsolationLevel)null, autoCommit);
    }
```

autoCommit可以开启为true



增

```java
    @Insert("insert into mybatis.user(name , pwd) values(#{name} , #{password})")
    int add(User user);
```

删

```java
    @Delete("delete from mybatis.user where id = #{id}")
    int deleteUserById(@Param("id") int id);s
```

改

```java
    @Update("update mybatis.user set name = #{name} , pwd = #{password} where id = #{id}")
    int updateUser(User user);
```

查

```java
    @Select("select * from mybatis.user")
    List<User> getUserList();

    //方法存在多个参数,所有的参数前面必须加上@Param()注解
    @Select("select * from user where id = #{id}")
    User getUserById(@Param("id") int id);
```

**注意:我们必须将接口绑定到核心配置文件中**



#### 关于@Param()注解

- 基本类型的参数或者String类型,需要加上
- 引用类型不需要加
- 如果只有一个基本类型的话,可以忽略,但是建议都加上
- 我们在SQL中引用的就是我们@Param()中设定的属性



**#{} ${}区别**

#{}可以防SQL注入

${}直接进行拼接不能防止SQL注入





### 9.Lombok

```java
Project Lombok is a javalibrary that automatically plugs into your editor and build tools, spicing up your java.
Never write another getter or equals method again , with one annotation your class has a fully featured builder , Automate your logging variables , and much more.
```



使用步骤

1. 在IDEA中安装Lombok插件

2. 在项目中导入lombok的jar包

   ```xml
   <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
   <dependency>
       <groupId>org.projectlombok</groupId>
       <artifactId>lombok</artifactId>
       <version>1.18.12</version>
   </dependency>
   ```

3. 在实体类加上注解

   ```java
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   ```



可以使用的注解:

- @Getter and @Setter
- @FieldNameConstants
- @ToString
- @EqualsAndHashCode
- @AllArgsConstructor, @RequiredArgsConstructor and @NoArgsConstructor
- @Log, @Log4j, @Log4j2, @Slf4j, @XSlf4j, @CommonsLog, @JBossLog, @Flogger, @CustomLog
- @Data
  - 无参构造
  - getter
  - setter
  - toString
  - hashcode
  - equals
- @Builder
- @SuperBuilder
- @Singular
- @Delegate
- @Value
- @Accessors
- @Wither
- @With
- @SneakyThrows
- @val
- @var
- experimental @var
- @UtilityClass
- @ExtensionMethod (Experimental, activate manually in plugin settings)





### 10.复杂查询

#### 多对一处理

```mysql
CREATE TABLE `teacher`(
	`id` INT(10) PRIMARY KEY,
	`name` VARCHAR(30) DEFAULT NULL
) ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO teacher(`id` , `name`) VALUES(1,"秦老师");

CREATE TABLE `student`(
	`id` INT(10) PRIMARY KEY,
	`name` VARCHAR(30) DEFAULT NULL,
	`tid` INT(10) DEFAULT NULL,
	KEY `fktid` (`tid`),
	CONSTRAINT `fktid` FOREIGN KEY (`tid`) REFERENCES `teacher` (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO `student` (`id` , `name` , `tid`) VALUES ("1" , "小赵" , "1"),("2" , "小钱" , "1"),("3" , "小孙" , "1"),("4" , "小李" , "1"),("5" , "小周" , "1");
```





#### 测试环境搭建

1. 导入lombok
2. 新建实体类Teacher,Student
3. 建立Mapper接口
4. 建理Mapper.xml文件
5. 在核心配置文件中绑定注册我们的Mapper接口或者文件
6. 测试查询

```mysql
select s.id , s.name , t.name from student s , teacher t where s.id = t.id
```



##### 按照查询嵌套处理

```xml
<select id="getStudent" resultMap="StudentTeacher">
        select * from mybatis.student;
    </select>
    
    <resultMap id="StudentTeacher" type="Student">
        <result property="id" column="id" />
        <result property="name" column="name" />
        <!-- 复杂的属性,我们需要单独处理 对象: association 集合: collection -->
        <association property="teacher" column="tid" javaType="Teacher" select="getTeacher" />
    </resultMap>

    <select id="getTeacher" resultType="Teacher">
        select * from teacher where id = #{id}
    </select>
```





##### 按照结果嵌套处理

```xml
    <select id="getStudent2" resultMap="StudentTeacher2">
        select s.id sid , s.name sname , t.name tname
        from student s , teacher t
        where s.tid = t.id
    </select>
    
    <resultMap id="StudentTeacher2" type="Student">
        <result property="id" column="sid" />
        <result property="name" column="sname" />
        <association property="teacher" javaType="Teacher" >
            <result property="name" column="tname" />
        </association>
    </resultMap>
```





#### 一对多处理

比如一个老师对应多个学生

对于老师而言,就是一对多关系



Student.java

```java
public class Student {

    private int id;

    private String name;

    //学生需要关联一个老师
    private int tid;

}
```

Teacher.java

```java
@Data
public class Teacher {

    private int id;

    private String name;

    private List<Student> students;

}
```

##### 按照结果嵌套处理

TeacherMapper.xml

```xml
<select id="getTeacher" resultMap="TeacherStudent">
        select s.id sid , s.name sname , t.name tname , t.id tid
        from student s , teacher t
        where s.tid = t.id and t.id = #{tid}
    </select>
    
    <resultMap id="TeacherStudent" type="Teacher">
        <result property="id" column="tid"/>
        <result property="name" column="tname"/>
        <!-- javaType = "" 指定属性的类型
                集合中的泛型信息,我们使用ofType获取
         -->
        <collection property="students" ofType="Student">
            <result property="id" column="sid" />
            <result property="name" column="sname" />
            <result property="tid" column="tid" />
        </collection>
    </resultMap>
```

结果

```java
Teacher(id=1, 
        name=秦老师, 
        students=[Student(id=1, name=小赵, tid=1), 
                  Student(id=2, name=小钱, tid=1), 
                  Student(id=3, name=小孙, tid=1), 
                  Student(id=4, name=小李, tid=1), 
                  Student(id=5, name=小周, tid=1)])
```



##### 按照查询嵌套处理

```xml
    <select id="getTeacher2" resultMap="TeacherStudent2">
        select * from mybatis.teacher where id = #{tid}
    </select>
    
    <resultMap id="TeacherStudent2" type="Teacher">
        <collection property="students" javaType="ArrayList" ofType="Student" select="getStudentByTeacherId" column="id" />
    </resultMap>

    <select id="getStudentByTeacherId" resultType="Student">
        select * from mybatis.student where tid = #{tid}
    </select>
```







##### 小结

1.关联--association [多对一]

2.集合--collection [一对多]

3.javaType 用来指定实体类中属性的类型

4.ofType 用来指定映射到List或者集合中的pojo类型,泛型中的约束类型



注意点:

- 保证Sql的可读性,尽量保证通俗易懂
- 注意一对多和多对一中,属性名和字段的问题
- 如果问题不好排查错误,可以使用日志,建议使用Log4j





面试高频

- Mysql引擎
- InnoDB底层原理
- 索引
- 索引优化







### 11.动态SQL

==**所谓的动态SQL,本质还是SQL语句,只是我们可以在SQL层面,去执行一个逻辑代码**==

**什么是动态SQL**

动态SQL就是指根据不同的条件生成不同的SQL语句

```xml
如果你之前用过 JSTL 或任何基于类 XML 语言的文本处理器，你对动态 SQL 元素可能会感觉似曾相识。在 MyBatis 之前的版本中，需要花时间了解大量的元素。借助功能强大的基于 OGNL 的表达式，MyBatis 3 替换了之前的大部分元素，大大精简了元素种类，现在要学习的元素种类比原来的一半还要少。

if
choose (when, otherwise)
trim (where, set)
foreach
```





#### 搭建环境

```mysql
CREATE TABLE `blog`
(
	`id` VARCHAR(50) NOT NULL COMMENT`博客id`,
	`title` VARCHAR(100) NOT NULL COMMENT`博客标题`,
	`author` VARCHAR(30) NOT NULL COMMENT`博客作者`,
	`create_time` DATETIME NOT NULL COMMENT`创建时间`,
	`views` INT(30) NOT NULL COMMENT`浏览量`
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```



创建一个基础工程

1. 导包

2. 编写配置文件

3. 编写实体类

   ```java
   public class Blog {
   
       private int id;
   
       private String title;
   
       private String author;
   
       private Date createTime;
   
       private int views;
   
   }
   ```

4. 编写实体类对应的Mapper借口和Mapper.xml



```java
@SuppressWarnings("all") //抑制警告
```



获取随机ID

```java
public class IDUtils {

    public static String getId(){
        return UUID.randomUUID().toString().replaceAll("-" , "");
    }

}
```





#### IF

```xml
    <select id="queryBlogIF" parameterType="map" resultType="blog">
        select * from mybatis.blog where 1=1
        <if test="title != null">
            and title = #{title}
        </if>
        <if test="author != null">
            and blog.author = #{author}
        </if>
    </select>
```





#### choose (when, otherwise)

有时候，我们不想使用所有的条件，而只是想从多个条件中选择一个使用。针对这种情况，MyBatis 提供了 choose 元素，它有点像 Java 中的 switch 语句。

```xml
    <select id="queryBlogChoose" parameterType="map" resultType="blog">
        select * from mybatis.blog
        <where>
            <choose>
                <when test="title != null">
                    title = #{title}
                </when>
                <when test="author != null">
                    and author = #{author}
                </when>
                <otherwise>
                    and views = #{views};
                </otherwise>
            </choose>
        </where>
    </select>
```





#### trim (where, set)

##### where

*where* 元素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，*where* 元素也会将它们去除。

```xml
select * from mybatis.blog
        <where>
            <if test="title != null">
                title = #{title}
            </if>
            <if test="author != null">
                and blog.author = #{author}
            </if>
        </where>
```



##### set

```xml
    <update id="updateBlog" parameterType="map">
        update mybatis.blog
        <set>
            <if test="title != null">
                title = #{title},
            </if>

            <if test="author != null">
                    author = #{author}
            </if>
        </set>
        where id = #{id};
    </update>
```





#### Foreach

```mysql
select * from user where 1 = 1 and (id = 1 or id = 2 or id = 3)

	<foreach item="id" index="index" collection="ids"
		open="(" separator="or" close=")">
		#{id}
	</foreach>
	
(id=1 or id=2 or id=3)
```

动态 SQL 的另一个常见使用场景是对集合进行遍历（尤其是在构建 IN 条件语句的时候）。

*foreach* 元素的功能非常强大，它允许你指定一个集合，声明可以在元素体内使用的集合项（item）和索引（index）变量。它也允许你指定开头与结尾的字符串以及集合项迭代之间的分隔符。这个元素也不会错误地添加多余的分隔符，看它多智能！

你可以将任何可迭代对象（如 List、Set 等）、Map 对象或者数组对象作为集合参数传递给 *foreach*。当使用可迭代对象或者数组时，index 是当前迭代的序号，item 的值是本次迭代获取到的元素。当使用 Map 对象（或者 Map.Entry 对象的集合）时，index 是键，item 是值。

![mybatis10](E:\files\JAVA\笔记\2021\resources\mybatis\mybatis10.png)

==**所谓动态SQL,本质还是SQL语句,只是我们可以在SQL层面,去执行一个逻辑代码**==



#### SQL片段

有的时候,我们可能会将一些公共的部分抽取出来,方便复用

1. 使用SQL标签抽取公共的部分

   ```xml
       <sql id="if-title-author">
           <if test="title != null">
               title = #{title}
           </if>
   
           <if test="author != null">
               and author = #{author}
           </if>
       </sql>
   ```

2. 在需要使用的地方使用Include标签应用

   ```xml
       <select id="queryBlogIF" parameterType="map" resultType="blog">
           select * from mybatis.blog
           <where>
               <include refid="if-title-author"></include>
           </where>
       </select>
   ```



注意事项:

- 最好基于单表来定义SQL片段
- 不要存在where标签



==**动态SQL就是在拼接SQL语句,我们只要保证SQL的正确性,按照SQL的格式,去排列组合就可以了**==



建议:

- 先在Mysql中写出完整的SQL,再对应地去修改成为我们的动态SQL



### 12.缓存

#### 简介

```
查询	:	连接数据库,耗资源
	一次查询的结果,暂存在一个可以直接取到的地方	- 内存:缓存
	
再次查询相同数据的时候,直接走缓存,不用走数据库
```



1. 什么是缓存
   - 存在内存中的临时数据
   - 将用户经常查询的数据放在缓存(内存)中,用户去查询数据就不用从磁盘上(关系型数据库数据文件)查询,从缓存中查询,从而提高查询效率,解决了高并发系统的性能问题
2. 为什么使用缓存
   - 减少和数据库的交互次数,减少系统的开销,提高系统效率
3. 什么样的数据能使用缓存?
   - 经常查询并且不经常改变的数据



#### Mybatis缓存

- MyBatis包含一个非常强大的查询缓存特性,它可以非常方便地定制和配置缓存.缓存可以极大的提升查询效率
- MyBatis系统中默认定义了两级缓存:**一级缓存**和**二级缓存**
  - 默认情况下,只有一级缓存开启.(SqlSession级别的缓存,也成为本地缓存)
  - 二级缓存需要手动开启和配置,它是基于namespace级别的缓存
  - 为了提高扩展性,MyBatis定义了缓存借口Cache.我们可以通过实现Cache接口来自定义二季缓存



#### 一级缓存

- 一级缓存也叫本地缓存: SqlSession级别
  - 与数据库同一次会话期间查询到的数据会放在本地缓存中.
  - 以后如果需要获取相同的数据,直接从缓存中拿,没必要再去查询数据库



测试步骤:

1. 开启日志

2. 测试在一个sqlSession中查询两次记录

   ```java
       @Test
       public void queryUserById(){
           SqlSession sqlSession = MybatisUtils.getSqlSession();
           UserMapper mapper = sqlSession.getMapper(UserMapper.class);
   
           User user = mapper.queryUserById(1);
           System.out.println(user);
   
           User user2 = mapper.queryUserById(1);
           System.out.println(user2);
   
           System.out.println(user == user2);
   
           sqlSession.close();
       }
   ```

   

3. 查看日志输出

![mybatis11](E:\files\JAVA\笔记\2021\resources\mybatis\mybatis11.png)



缓存失效的情况:

1. 增删改操作,可能会改变原来的数据,所以必定会刷新缓存!

   ![mybatis12](E:\files\JAVA\笔记\2021\resources\mybatis\mybatis12.png)

2. 查询不同的东西

   ![mybatis13](E:\files\JAVA\笔记\2021\resources\mybatis\mybatis13.png)

   ![mybatis14](E:\files\JAVA\笔记\2021\resources\mybatis\mybatis14.png)

3. 查询不同的Mapper.xml

4. 手动清理缓存



小结:一级缓存默认是开启的,只在一次SqlSession中有效,也就是拿到连接到关闭连接这个时间段

一级缓存就是一个Map



#### 二级缓存

- 二级缓存也叫全局缓存,一级缓存作用域太低了,于是诞生了二级缓存
- 基于namespace级别的缓存,一个名称空间,对应一个二级缓存
- 工作机制
  - 一个会话查询一条数据,这个数据就会被放在当前会话的一级缓存中
  - 如果当前会话被关闭了,这个会话对应的一级缓存就没了;但是我们想要的是,会话关闭了,一级缓存中的数据被保存到二级缓存中;
  - 新的会话查询信息,就可以从二级缓存中获取内容;
  - 不同的mapper查出的数据会放在自己对应的缓存(map)中



步骤:

1. 开启缓存

   ```xml
           <!--开启全局缓存-->
           <setting name="cacheEnabled" value="true"/>
   ```

2. 在要使用二级缓存的Mapper中开启

       <!--在当前Mapper.xml中使用二级缓存-->
       <cache eviction="FIFO"
               flushInterval="60000"
               size="512"
               readOnly="true"/>
   也可以自定参数

3. 测试

   1. 问题:我们需要将实体类序列化,否则就会报错

      ```java
      Cause: java.io.NotSerializableException: com.nidie.pojo.User
      ```



小结:

- 只要开启了二级缓存,只要在同一个Mapper下就有效
- 所有的数据都会先放在一级缓存中
- 只有当会话提交,或者关闭的时候,才会提交到二级缓存中





#### 缓存原理

缓存顺序:

1. 先看二级缓存中有没有
2. 再看一级缓存中有没有
3. 都没有,查询数据库



![mybatis15](E:\files\JAVA\笔记\2021\resources\mybatis\mybatis15.png)







#### 自定义缓存-ehcache

Ehcache是一种广泛使用的开源Java分布式缓存.主要面向通用缓存



导包

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis.caches/mybatis-ehcache -->
<dependency>
    <groupId>org.mybatis.caches</groupId>
    <artifactId>mybatis-ehcache</artifactId>
    <version>1.1.0</version>
</dependency>
```



在mapper中指定使用我们的ehcache缓存实现

```xml
    <cache type="org.mybatis.caches.ehcache.EhcacheCache" />
```





ehcache.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
        updateCheck="false">

<diskStore path="./tmpdir/Tmp_EhCache" />

<defaultCache
eternal="false"
maxElementsInMemory="10000"
        overflowToDisk="false"
        diskPersistent="false"
        timeToIdleSeconds="1800"
        timeToLiveSeconds="259200"
        memoryStoreEvictionPolicy="LRU"/>

<cache
name="cloud_user"
eternal="false"
maxElementsInMemory="5000"
        overflowToDisk="false"
        diskPersistent="false"
        timeToIdleSeconds="1800"
        timeToLiveSeconds="1800"
        memoryStoreEvictionPolicy="LRU"/>

</ehcache>
```







## Spring5

### 1.简介

==**Spring是一个轻量级的控制反转(IoC)和面向切面(AOP)的容器框架**==



- SSH: Struct2 + Spring + Hibernate
- SSM: SpringMVC + Spring + Mybatis



maven依赖

```xml
        <!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.2.0.RELEASE</version>
        </dependency>
        
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.2.0.RELEASE</version>
        </dependency>
```





#### 优点

- Spring是一个开源的免费的框架(容器)
- Spring是一个轻量级的、非入侵式的框架
- 控制反转(IoC),面向切面编程(AOP)
- 支持事务的处理,对框架整合的支持



==总结:Spring就是一个轻量级的控制反转(IoC)和面向切面编程(AOP)的框架==



##### Spring的组成

现代化的java开发!说白就是基于Spring的开发

![1628730178(1)](E:\files\JAVA\笔记\2021\resources\Spring\1628730178(1).png)



###### 拓展

- Spring Boot
  - 一个快速开发的脚手架
  - 基于SpringBoot可以快速的开发单个微服务
  - 约定大于配置
- Spring Cloud
  - SpringCloud是基于SpringBoot实现的



因为现在大多数公司都在使用SpringBoot进行快速开发,学习SpringBoot的前提,需要完全掌握Spring及SpringMVC

SpringMVC起到承上启下作用



弊端:发展了太久之后,违背了原来的理念!配置十分繁琐,人称:"配置地狱"



### 2.IoC理论推导

1. UserDao接口
2. UserDaoImpl实现类
3. UsersService业务接口
4. UserServiceImpl业务实现类



在我们之前的业务中,用户的需求可能会影响我们原来的代码,我们需要根据用户的需求去修改源代码!如果程序代码量十分大,修改一次的成本代价十分昂贵



我们使用一个Set接口实现,已经发生了革命性的变化

```java
public class UserServiceImpl implements UserService {

    private UserDao userDao;

    public void setUserDao(UserDao userDao){
        this.userDao = userDao;
    }
}
```



- 之前,程序是主动创建对象!控制权在程序员手上!
- 使用了set注入后,程序不再具有主动性,而是变成了被动的接收对象



这种思想,从本质上解决了问题,我们程序员不用再去管理对象的创建了.系统的耦合性大大地降低,可以更加专注的在业务的实现上,这是IoC的原型



#### IoC本质

**控制反转IoC(Inversion of Control),是一种设计思想,DI(依赖注入)是实现IoC的一种方法,**也有人认为DI只是IoC的另一种说法.没用IoC的程序中,我们使用面向对象编程,对象的创建与对象间的依赖关系完全硬编码在程序中,对象的创建由程序自己控制,控制反转后将对象的创建转移给第三方,个人认为所谓控制反转就是:获得依赖对象的方式反转了

![spring2](E:\files\JAVA\笔记\2021\resources\Spring\spring2.png)

**IoC是Spring框架的核心内容**,使用多种方式完美地实现了IoC,可以使用XML配置,也可以使用注解,新版本的Spring也可以零配置实现IoC.

Spring容器在初始化时先读取配置文件,根据配置文件或元数据创建与组织对象存入容器中,程序使用时再从IoC容器中取出需要的对象



采用XML方式配置Bean的时候,Bean的定义信息是和实现分离的,而采用注解的方式可以把两者合为一体,Bean的定义信息直接以注解的形式定义在实现类中,从而达到了零配置的目的

**控制反转是一种通过描述(XML或注解)并通过第三方去生产或获取特定对象的方式.在spring中实现控制反转的是IoC容器,其实现的方法是依赖注入(Dependency Injection,DI)**





### 3.HelloSpring

**beans.xml创建**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```

#### 思考问题?

- Hello对象是谁创建的?

  hello对象是由Spring创建的

- Hello对象的属性是怎么设置的

  hello对象的属性是由Spring容器设置的

这个过程就叫控制反转:
控制:谁来控制对象的创建,传统应用程序的对象是由程序本身控制创建的,使用Spring后,对象是由Spring来创建对的

反转:程序本身不创建对象,而编程被动的接收对象

依赖注入:就是利用set方法来进行注入的

IoC是一种编程思想,由主动的编程编程被动的接收

可以通过newClassPathXmlApplicationContext去浏览一下底层源码

IoC:对象由Spring来创建,管理,装配





### 4.IoC创建对象的方式

1. 使用无参构造创建对象(默认)

2. 假设我们要使用有参构造创建

   1. 下标赋值

      ```xml
          <!--  第一种,下标赋值  -->
          <bean id="user" class="com.nidie.pojo.User">
              <constructor-arg index="0" value="你爹" />
          </bean>
      ```

   2. 通过类型创建

      ```xml
          <!--  第二种:通过类型创建,不建议使用!  -->
          <bean id="user" class="com.nidie.pojo.User">
              <constructor-arg type="java.lang.String" value="你爹" />
          </bean>
      ```

   3. 通过参数名

      ```xml
          <!--  第三种:直接通过参数名来设置  -->
          <bean id="user" class="com.nidie.pojo.User">
              <constructor-arg name="name" value="你爹123" />
          </bean>
      ```

   总结:在配置文件加载的时候,容器中的对象就已经初始化了

   

   

   ### 5.Spring配置

   #### 1.别名

   ```xml
    <!--  别名,如果添加了别名,我们也可以使用别名获取到这个对象  -->
       <alias name="user" alias="u" />
	```
   

   

   
#### 2.Bean的配置

```xml
       <!--
        id:bean的唯一标识符,也就是相当于我们学的对象名
           class: bean对象所定义的权限定名: 包名+类名
        name: 别名,而且name更高级,可以同时取多个别名
       -->
    <bean id="userT" class="com.nidie.pojo.UserT" name="user2 u2,u3;u4">
           <property name="name" value="你爹" />
    </bean>
   
    <!--  别名,如果添加了别名,我们也可以使用别名获取到这个对象  -->
       <alias name="user" alias="u" />
```





   #### 3.import

   这个import,一般用于团队开发使用,他可以将多个配置文件,倒入合并为一个

   架设,现在项目中有多个人开发,这三个人负责不同的类的开发,不同的类需要注册在不同的bean中,我们可以利用import,将所有人的beans.xml合并为一个总的

   - 张三

   - 李四

   - 王五

   - applicationContext.xml

     ```xml
         <import resource="beans.xml" />
         <import resource="beans2.xml" />
         <import resource="beans3.xml" />
     ```



最后使用的时候,直接使用总的配置就可以了

   



 

### ==5.依赖注入==

#### 构造器注入





#### **Setter注入[重点]**

- 依赖注入: Setter注入
  - 依赖: bean对象的创建依赖于容器
  - 注入: bean对象中的所有属性,由容器来注入



**环境搭建**

1.复杂类型

```java
package com.nidie.pojo;

public class Address {

    private String address;

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

}

```

2.真实测试对象

```java
package com.nidie.pojo;

import java.util.*;

public class Student {

    private String name;

    private Address address;

    private String[] books;

    private List<String> hobbies;

    private Map<String , String> card;

    private Set<String> games;

    private Properties info;

    private String wife;

}

```

3.beans.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="student" class="com.nidie.pojo.Student" >
        <!--    第一种:普a通值注入,value    -->
        <property name="name" value="你爹" />
    </bean>

</beans>
```

4.测试类

```java
	@Test
    public void TestStudent(){
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        Student student = (Student) context.getBean("student");
        System.out.println(student.toString());
    }
```

5.完善注入信息

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="address" class="com.nidie.pojo.Address" >
        <property name="address" value="America" />
    </bean>

    <bean id="student" class="com.nidie.pojo.Student" >
        <!--    第一种:普a通值注入,value    -->
        <property name="name" value="你爹" />

        <!--    第二种:Bean注入 , ref    -->
        <property name="address" ref="address" />

        <!--    数组注入,ref    -->
        <property name="books">
            <array>
                <value>西游记</value>
                <value>红楼梦</value>
                <value>水浒传</value>
                <value>三国演义</value>
            </array>
        </property>

        <!-- List注入 -->
        <property name="hobbies">
            <list>
                <value>踢足球</value>
                <value>lol</value>
                <value>敲代码</value>
            </list>
        </property>

        <!-- Map注入 -->
        <property name="card">
            <map>
                <entry key="身份证" value="431222200103135650" />
                <entry key="银行卡" value="6228482429439027177" />
            </map>
        </property>

        <!-- Set注入 -->
        <property name="games">
            <set>
                <value>lol</value>
                <value>COC</value>
                <value>CF</value>
            </set>
        </property>

        <!--    null值注入    -->
        <property name="wife">
            <null />
        </property>

        <property name="info">
            <props>
                <prop key="driver">123456789</prop>
                <prop key="url">男</prop>
                <prop key="username">root</prop>
                <prop key="password"/>
            </props>
        </property>
    </bean>

</beans>
```

6.测试结果

```java
    /*
    * Student{name='你爹',
    * address=Address{address='America'},
    * books=[西游记, 红楼梦, 水浒传, 三国演义],
    * hobbies=[踢足球, lol, 敲代码],
    * card={
    *   身份证=431222200103135650,
    *   银行卡=6228482429439027177},
    * games=[lol, COC, CF],
    * info={    password=,
    *   url=男,
    *   driver=123456789,
    *   username=root
    * },
    * wife='null'}
     * */
```







#### 拓展方式注入

##### p命名空间注入

1.实体类

```java
package com.nidie.pojo;

public class User {

    private String name;

    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

}

```



2.配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--  p命名空间注入,可以直接注入属性的值:property  -->
    <bean id="user" class="com.nidie.pojo.User" p:name="你爹" p:age="20" />

</beans>
```



   3.测试

```java
    @Test
    public void TestUser(){
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("userBeans.xml");
        User user = context.getBean("user" , User.class);
        System.out.println(user.toString());
    }
```



#####    C命名空间注入

1.配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--  c命名空间注入,可以通过构造器注入:construt-args  -->
    <bean id="user2" class="com.nidie.pojo.User" c:name="你爷爷" c:age="18"  />

</beans>
```



2.测试运行

```java
    @Test
	public void TestUser(){
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("userBeans.xml");
        User user = context.getBean("user2" , User.class);
        System.out.println(user.toString());
    }
```





注意点: p命名空间和c命名空间不能直接使用,需要导入xml约束

   ```xml
xmlns:p="http://www.springframework.org/schema/p"
xmlns:c="http://www.springframework.org/schema/c"
   ```



  

#### Bean的作用域

![1628939777(1)](E:\files\JAVA\笔记\2021\resources\Spring\1628939777(1).jpg)

1. 单例模式

   ```xml
   bean id="user2" class="com.nidie.pojo.User" c:name="你爷爷" c:age="18" scope="singleton"/>
   ```

2. 原型模式:每次从容器中get的时候,都会产生一个新的对象

   ```xml
   <bean id="user2" class="com.nidie.pojo.User" c:name="你爷爷" c:age="18" scope="prototype" />
   ```

3. 其余的request、session、application、这些只能在Web开发中使用到





###    ==6.Bean的自动装配==

- 自动装配是Spring满足bean依赖的一种方式

- Spring会在上下文中自动寻找,并自动给Bean装配属性



在Spring中有三种装配的方式

1. 在xml中显示地配置
2. 在java中显示配置
3. **隐式的自动装配**[重要]



#### 配置

环境搭建

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="cat" class="com.nidie.pojo.Cat" />
    <bean id="dog" class="com.nidie.pojo.Dog" />

    <bean id="person" class="com.nidie.pojo.Person" >
        <property name="name" value="你爹" />
        <property name="dog" ref="dog" />
        <property name="cat" ref="cat" />
    </bean>

</beans>
```

pojo实体类

```java
package com.nidie.pojo;

public class Person {

    private Cat cat;

    private Dog dog;

    private String name;

    public Cat getCat() {
        return cat;
    }

    public void setCat(Cat cat) {
        this.cat = cat;
    }

    public Dog getDog() {
        return dog;
    }

    public void setDog(Dog dog) {
        this.dog = dog;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Person{" +
                "cat=" + cat +
                ", dog=" + dog +
                ", name='" + name + '\'' +
                '}';
    }
}

```

```java
package com.nidie.pojo;

public class Dog {

    public void bark(){
        System.out.println("汪~");
    }

}

```

```java
package com.nidie.pojo;

public class Cat {

    public void bark(){
        System.out.println("喵~");
    }

}

```



#### byName自动装配

```xml
    <!--  byName:会自动在容器上下文中,和自己对象set方法后面的值对应的beanId  -->
    <bean id="person" class="com.nidie.pojo.Person" autowire="byName">
        <property name="name" value="你爹" />
    </bean>
```

#### byType自动装配

```xml
<bean class="com.nidie.pojo.Cat" />
<bean class="com.nidie.pojo.Dog" />

<!--
	byName:会自动在容器上下文中,和自己对象set方法后面的值对应的beanId
	byType: 会自动在容器上下文查找,和自己对象属性类型相同的bean byType必须保证类型全局唯一
-->
<bean id="person" class="com.nidie.pojo.Person" autowire="byType">
	<property name="name" value="你爹" />
</bean>
```

byType可以不用写id

小结:

- byName的时候,需要保证所有bean的id唯一,并且这个bean需要和自动注入的属性的set方法的值一致
- byType的时候,需要保证所有bean的class唯一,并且这个bean需要和自动注入的属性的类型一致



#### 使用注解实现自动装配

jdk1.5支持的注解,Spring2.5就支持注解了

要使用注解须知:
1.导入约束

2.==配置注解的支持：<<context:annotation-config/>>==

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```



**@Autowired**

直接在属性上使用即可,也可以在set方法上使用

使用Autowire我们可以不用编写Set方法了,前提你这个自动装配的属性在IoC(Spring)容器存在,且符合名字byName

科普

```xml
@Nullable 字段标记了这个注解,说明这个字段可以为null;
```

```java
public @interface Autowired {
    boolean required() default true;
}
```

测试代码

```java
    //如果显示定义了Autowired的required属性为false,说明这个对象可以为null,否则不能为空
    @Autowired(required = false)
    private Cat cat;
```



如果@Autowired自动装配的环境比较复杂,自动装配无法通过一个注解[@Autowired]完成的时候,我们可以使用@Qualifier(value="xxx")去配合@Autowired的使用,指定一个唯一的bean对象注入

```java
public class Person {

    @Autowired
    @Qualifier(value="cat111")
    private Cat cat;

    @Autowired
    @Qualifier(value="dog222")
    private Dog dog;
    private String name;
}
```



**@Reource注解**

```java
public class Person {

    @Resource(name = "cat2")
    private Cat cat;

    @Resource
    private Dog dog;
}
```

小结:
@Resource和@Autowired的区别:

- 都是用来自动装配的,都可以放在属性字段上
- @Autowired默认通过byName的方式实现[常用]
- @Reource默认通过byName的方式实现,如果找不到名字,则通过byType实现.如果两个都找不到的情况下就报错
- 执行顺序不同: @Autowired先通过byType的方式实现.@Resource默认通过byName的方式实现





### ==7.使用注解开发==

在Spring4之后,要使用注解开发,必须要保证aop的包导入了

使用注解需要导入context约束,增加注解的支持

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <!--  开启注解支持  -->
    <context:annotation-config/>

</beans>
```





1.bean

2.属性如何注入

```java
//等价于<bean id="user" class="com.nidie.pojo.User">

@Component
public class User {

    //相当于<property name="name" value="你爹" />
    public String name;

    @Value("你爹")
    public void setName(String name) {
        this.name = name;
    }

}
```

3.衍生的注解

​	@Component有几个衍生注解,我们在Web开发中,会按照mvc三层架构分层

- dao层[@Repository]

- service层 [@Service]

- controller层 [@Controller]

  这四个注解功能都是一样的,都是代表将某个类注册到Spring容器中,装备Bean

4.自动装配

- @Autowired:自动装配通过类型,名字

  ​	如果Autowired不能唯一自动装配上属性,则需要通过@Qualifier(value="xxx")

- @Nullable字段标记了这个注解,说明这个字段可以为null

- @Resource:自动配置通过名字.类型

- @Component:组件,放在类上,说明这个类被Spring管理了,就是bean!

5.作用域

```java
@Scope("prototype")
```

6.小结

xml与注解:

- xml更加万能,适用于任何场合,维护简单方便
- 注解 不是自己的类使用不了,维护相对复杂

xml与注解最佳实践:

- xml用来管理bean
- 注解只负责完成属性注入
- 我们在使用的过程中,只需要注意一个问题:必须让注解生效,就需要开启注解的支持

```xml
    <!--  指定要扫描的包,这个包下的注解就会生效  -->
    <context:component-scan base-package="com.nidie" />
    <context:annotation-config/>
```





### 8.使用Java的方式配置Spring

我们现在要完全不使用Spring的xml配置二路,全权交给Java来做

JavaConfig是Spring的一个子项目,在Spring4之后,它成为了一个核心功能



实体类

```java
//这里这个注解的意思,就是说明这个类被Spring接管了,注册到了容器中
@Component
public class User {

    private String name;

    public String getName() {
        return name;
    }

    @Value("你爹") //属性注入值
    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                '}';
    }

}

```

配置Config类

```java
//这个也会被Spring容器托管,注册到容器中,因为他本来就是一个@Component
//@Configuration代表这是一个配置类,和我们之前看的beans.xml相同
@Configuration
@ComponentScan("com.nidie.pojo")
public class NidieConfig {

    //注册一个bean,就相当于我们之前写的<bean>,
    //这个方法的名字,就相当于bean标签中的id属性
    //这个方法的返回值,就相当于bean标签中的class属性
    @Bean
    public User user(){
        return new User(); //就是返回要注入到bean的对象
    }

}

```

```java
@Configuration
@ComponentScan("com.nidie.pojo")
@Import(NidieConfig2.class)
public class NidieConfig2 {
}
```

测试类

```java
public class MyTest {

    @Test
    public void Test1(){
//        如果完全使用了配置类方式去做,我们就只能通过AnnotationConfigApplicationContext上下文来获取容器,通过配置类的class对象的加载
        ApplicationContext context = new AnnotationConfigApplicationContext(NidieConfig.class);
        User getUser = context.getBean("user" , User.class);
        System.out.println(getUser.getName());
    }

}
```



  这种纯Java的配置方式,在SpringBoot中随处可见

  

  

###   ==9. 代理模式==

为什么要学习代理模式?

因为这就是SpringAOP的底层(面向切面编程的底层实现)[SpringAOP和SpringMVC面试必问]

代理模式:

- 静态代理
- 动态代理

![image-20210815190950886](E:\files\JAVA\笔记\2021\resources\Spring\image-20210815190950886.png)



  

#### 静态代理

角色分析:

- 抽象角色:一般会使用接口或者抽象类来解决
- 真实角色:被代理的角色
- 代理角色:代理真实角色,代理真实角色后,我们一般会做一些附属操作
- 客户:访问代理对象的人



代码步骤:

1. 接口

   ```java
   //租房
   public interface Rent {
   
       public void rent();
   
   }
   ```

2. 真实角色

   ```java
   //房东
   public class Host implements Rent {
   
       public void rent() {
           System.out.println("房东要出租房子");
       }
   
   }
   ```

3. 代理角色

   ```java
   public class Proxy implements Rent {
   
       private Host host;
   
       public Proxy() {
       }
   
       public Proxy(Host h) {
           this.host = h;
       }
   
       public void rent() {
           seeHouse();
           host.rent();
           ht();
           fare();
       }
   
   //    看房
       public void seeHouse(){
           System.out.println("中介带你看房");
       }
   
   //    签合同
       public void ht(){
           System.out.println("签租赁合同");
       }
   
   //    收中介费
       public void fare(){
           System.out.println("收中介费");
       }
   
   }
   ```

4. 客户端访问代理角色

   ```java
   public class Client {
   
       @Test
       public void rent(){
           //房东要租房子
           Host host = new Host();
           //代理,中介帮房东租房子,但是代理角色一般会有一些附属操作
           Proxy proxy = new Proxy(host);
   
           //你不用面对房东,直接找中介租房
           proxy.rent();
       }
   
   }
   ```



代理模式的好处:

- 可以使真实角色的操作更加纯粹.不用去关注一些公共的业务
- 公共业务交给了代理角色,实现了业务的分工
- 公共业务发生扩展的时候,方便集中管理
- 一个动态代理类代理的是一个接口,一般就是对应的一类业务
- 一个动态代理类可以代理多个类,只要是实现了同一个接口即可

缺点:

- 一个真实角色就会产生一个代理角色;代码量会翻倍.开发效率会变低.



##### 加深理解

代码:spring-08-proxy

聊聊AOP

![image-20210815214124353](E:\files\JAVA\笔记\2021\resources\Spring\image-20210815214124353.png)





#### 动态代理

- 动态代理和静态代理角色一样
- 动态代理的代理类是动态生产的,不是程序员直接写好的
- 动态代理分为两大类:
  - 基于接口的动态代理:JDK的动态代理[我们在这里使用]
  - 基于类的动态代理: cdlib
  - java字节码实现: Javasist



需要了解两个类:Proxy 代理,InvacationHandler 调用处理程序



##### InvacationHandler





##### Proxy





### ==10.AOP==

#### 什么是AOP

AOP(Apect Oriented Programming)意为:面向切面编程,通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术.AOP是OOP的延续,是软件开发中的一个热点,也是Spring框架中的一个重要内容,是函数式编程的一种衍生范型.利用AOP可以对业务逻辑的各个部分进行隔离,从而使得业务逻辑各部分之间的耦合度降低,提高程序的可重用性,同时提高了开发效率

![1629292637(1)](E:\files\JAVA\笔记\2021\resources\Spring\1629292637(1).png)



#### AOP在Spring中的作用

==提供声明式业务;允许用户自定义切面==

- 横切关注点: 跨越应用程序多个模块的方法或功能.即是,与我们业务逻辑无关的,但是我们需要关注的部分,就是横切关注点.如日志,安全,缓存,事务等等...
- 切面(Aspect): 横切关注点被模块化的特殊对象.即,它是一个类
- 通知(Advice): 切面必须要完成的工作.即,它是类中的一个方法
- 目标(Target): 被通知对象
- 代理(Proxy): 向目标对象应用通知之后创建的对象
- 切入点(PointCut): 切面通知 执行的"地点"的定义
- 连接点(jointPoint): 与切入点匹配的执行点

![1629293039(1)](E:\files\JAVA\笔记\2021\resources\Spring\1629293039(1).png)

Spring AOP中,通过Advice定义横切逻辑,Spring中支持5种类的Advice

![1629293351(1)](E:\files\JAVA\笔记\2021\resources\Spring\1629293351(1).png)

即Aop在不改变原有代码的情况下,增加新功能



#### 使用Spring实现AOP

使用AOP织入,需要导入一个依赖包

```xml
<dependency>
	<groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.4</version>
</dependency>
```



##### 方式一: 使用Spring的API接口[主要是SpringAPI接口实现]

1.定义两个日志类:分别是在方法执行之前的日志和在方法执行之后的日志

```java
public class AfterLog implements AfterReturningAdvice {
    
    /*
        returnValue: 返回结果
    */
    public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
        System.out.println("执行了" + method.getName() + "返回结果为:" + returnValue);
    }

}
```

```java
public class BeforeLog implements MethodBeforeAdvice {

    //method: 要执行的目标对象的方法
    //args: 参数
    //target: 目标对象
    public void before(Method method, Object[] args, Object target) throws Throwable {
        System.out.println(target.getClass().getName() + "的" + method.getName() + "被执行了");
    }

}
```



2.定义接口和实现类

```java
public interface UserService {

    public void add();

    public void delete();

    public void update();

    public void select();

}
```

```java
public class UserServiceImpl implements UserService {

    public void add() {
        System.out.println("增加了一个用户");
    }

    public void delete() {
        System.out.println("删除了一个用户");
    }

    public void update() {
        System.out.println("修改了一个用户");
    }

    public void select() {
        System.out.println("查询了一个用户");
    }

}
```



==3.编写applicationContext.xml文件==

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--注册bean-->
    <bean id="userService" class="com.nidie.service.UserServiceImpl" />
    <bean id="beforeLog" class="com.nidie.log.BeforeLog" />
    <bean id="afterLog" class="com.nidie.log.AfterLog" />

    <!--配置aop:需要导入aop的约束-->
    <!--方式一:使用原生Spring API接口-->
    <aop:config>
        <!--    切入点:需要在哪个地方执行Spring的方法    -->
        <!--    expression:表达式,execution(要执行的位置: * * * * *)   -->
        <aop:pointcut id="pointcut" expression="execution(* com.nidie.service.UserServiceImpl.*(..))"/>

        <!--    执行环绕增加    -->
        <aop:advisor advice-ref="beforeLog" pointcut-ref="pointcut" />
        <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut" />
    </aop:config>

</beans>
```



4.测试

```java
public class MyTest {

    @Test
    public void test01(){
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        //动态代理代理的是接口
        UserService userService = context.getBean("userService", UserService.class);

        userService.add();
        userService.select();
    }

}
```

执行结果:

```java
com.nidie.service.UserServiceImpl的add被执行了
增加了一个用户
执行了add返回结果为:null
com.nidie.service.UserServiceImpl的select被执行了
查询了一个用户
执行了select返回结果为:null
```



BeforeLog和AfterLog分别在方法执行之前和之后执行,没有影响到程序的运行 





##### 方式二: 自定义类来实现AOP[主要是切面定义]

1.创建diy切面类和diy切面方法

```java
public class DiyPointCut {

    public void before(){
        System.out.println("========方法执行前========");
    }

    public void after(){
        System.out.println("========方法执行后========");
    }

}
```

2.配置ApplicationContext.xml文件

```xml
    <aop:config>
        <!--自定义切面, ref要引用的类-->
        <aop:aspect ref="diy">
            <!--切入点-->
            <aop:pointcut id="point" expression="execution(* com.nidie.service.UserServiceImpl.*(..))"/>
            <!--通知-->
            <aop:before method="before" pointcut-ref="point" />
            <aop:after method="after" pointcut-ref="point" />
        </aop:aspect>
    </aop:config>
```

使用

```xml
        <aop:before method="before" pointcut-ref="point" />
        <aop:after method="after" pointcut-ref="point" />
```
在方法前后插入日志

3.测试

```java
    @Test
    public void test02(){
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserService userService = context.getBean("userService", UserService.class);
        userService.select();
    }
```

结果:

```java
========方法执行前========
查询了一个用户
========方法执行后========
```





##### 方式三: 使用注解实现

1.配置AnnotationPointCut类

```java
//@Aspect 标注这个类是一个切面
@Aspect
@SuppressWarnings("all")
public class AnnotationPointCut {

    @Before("execution(* com.nidie.service.UserServiceImpl.*(..))")
    public void before(){
        System.out.println("=====方法执行前=====");
    }

    @After("execution(* com.nidie.service.UserServiceImpl.*(..))")
    public void after(){
        System.out.println("=====方法执行后=====");
    }

    //在环绕增强中,我们可以给定一个参数,代表我们要获取处理切入的点
    @Around("execution(* com.nidie.service.UserServiceImpl.*(..))")
    public void around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("环绕前");

        //获得签名
        Signature signature = joinPoint.getSignature();
        System.out.println("signature:" + signature);

        //执行方法
        Object proceed = joinPoint.proceed();

        System.out.println("环绕后");

        System.out.println(proceed);
    }

}
```



2.配置ApplicationContext.xml类

```xml
    <!--方式三:-->
    <bean id="annotationPointCut" class="com.nidie.diy.AnnotationPointCut" />
    <!--开启注解支持  JDK(默认) cglib(proxy-target-class)-->
    <aop:aspectj-autoproxy />
```



3.测试

```java
    @Test
    public void test03(){
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserService userService = context.getBean("userService", UserService.class);
        userService.select();
    }
```



4.输出结果

```java
环绕前
signature:void com.nidie.service.UserService.select()
=====方法执行前=====
查询了一个用户
环绕后
null
=====方法执行后=====
```









### 11.整合MyBatis

步骤:

1. 导入相关依赖

   - junit
   - mybatis
   - mysql数据库相关依赖
   - Spring相关的
   - AOP织入
   - mybatis-spring [新包]

2. 编写配置文件

   ```xml
       <dependencies>
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.12</version>
           </dependency>
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
               <version>5.1.47</version>
           </dependency>
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis</artifactId>
               <version>3.5.2</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-webmvc</artifactId>
               <version>5.1.9.RELEASE</version>
           </dependency>
           <!--Spring操作数据库,还需要一个spring-jdbc-->
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-jdbc</artifactId>
               <version>5.1.9.RELEASE</version>
           </dependency>
           <dependency>
               <groupId>org.aspectj</groupId>
               <artifactId>aspectjweaver</artifactId>
               <version>1.8.13</version>
           </dependency>
           <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis-spring</artifactId>
               <version>2.0.2</version>
           </dependency>
       </dependencies>
   ```

3. 测试



#### Mybatis

1. 编写实体类

   ```java
   /**
    * @author nidie
    */
   public class User {
   
       private int id;
   
       private String name;
   
       private String pwd;
   
       public User() {
       }
   
       public User(int id, String name, String pwd) {
           this.id = id;
           this.name = name;
           this.pwd = pwd;
       }
   
       public int getId() {
           return id;
       }
   
       public void setId(int id) {
           this.id = id;
       }
   
       public String getName() {
           return name;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
       public String getPwd() {
           return pwd;
       }
   
       public void setPwd(String pwd) {
           this.pwd = pwd;
       }
   
       @Override
       public String toString() {
           return "User{" +
                   "id=" + id +
                   ", name='" + name + '\'' +
                   ", pwd='" + pwd + '\'' +
                   '}';
       }
   
   }
   ```

2. 编写核心配置文件

   ```xml
   <?xml version = "1.0" encoding = "UTF-8" ?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
       
       <typeAliases>
           <package name="com.nidie.pojo"/>
       </typeAliases>
   
       <environments default="development">
   
           <environment id="development">
               <transactionManager type="JDBC" />
               <dataSource type="POOLED">
                   <property name = "driver" value = "com.mysql.jdbc.Driver"/>
                   <property name = "url" value = "jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8" />
                   <property name = "username" value = "root" />
                   <property name = "password" value = "" />
               </dataSource>
           </environment>
   
       </environments>
   
       <mappers>
           <mapper class="com.nidie.mapper.UserMapper" />
       </mappers>
   
   </configuration>
   ```

3. 编写接口

   ```java
   /**
    * @author nidie
    */
   public interface UserMapper {
   
       List<User> queryUsers();
   
   }
   ```

4. 编写Mapper.xml

   ```xml
   <?xml version = "1.0" encoding = "UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   
   <mapper namespace="com.nidie.mapper.UserMapper">
   
       <select id="queryUsers" resultType="user" >
           select * from mybatis.user;
       </select>
   
   </mapper>
   ```

5. 测试

   ```java
       @Test
       public void testQueryUsers() {
           SqlSession sqlSession = null;
           try {
               sqlSession = new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("mybatis-config.xml")).openSession(true);
           } catch (IOException e) {
               e.printStackTrace();
           }
   
           UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
           for (User queryUser : userMapper.queryUsers()) {
               System.out.println(queryUser);
           }
   
           sqlSession.close();
       }
   
   }
   ```



#### MyBatis-Spring

##### Spring整合Mybatis的方法一

1.用Spring配置数据源

spring-dao.xml

```xml
    <bean id="datasource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8" />
        <property name="username" value="root" />
        <property name="password" value="" />
    </bean>
```

配置SqlSessionFactory

```xml
    <!--sqlSessionFactory-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="datasource" />
        <!--绑定Mybatis配置文件-->
        <property name="configLocation" value="classpath:mybatis-config.xml" />
<!--        <property name="typeAliases" value="com.nidie.pojo.User" />-->
        <property name="mapperLocations" value="classpath:com/nidie/mapper/UserMapper.xml" />
    </bean>
```

注册SqlSessionTemplate

```xml
    <!--SqlSessionTemplate: 就是我们使用的SqlSession-->
    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate" >
        <!--只能通过构造器注入SqlSessionFactory , 因为没用setter方法-->
        <constructor-arg index="0" ref="sqlSessionFactory" />
    </bean>
```

给接口写实现类

在实现类中添加SqlSessionTemplate对象

```java
/**
 * @author nidie
 */
public class UserMapperImpl implements UserMapper {

    //我们的所有操作,都使用SqlSession来执行,在原来.现在都使用SqlSessionTemplate
    private SqlSessionTemplate sqlSessionTemplate;

    public UserMapperImpl() {
    }

    public UserMapperImpl(SqlSessionTemplate sqlSessionTemplate) {
        this.sqlSessionTemplate = sqlSessionTemplate;
    }

    public List<User> queryUsers() {
        UserMapper mapper = sqlSessionTemplate.getMapper(UserMapper.class);
        return mapper.queryUsers();
    }

    public void setSqlSessionTemplate(SqlSessionTemplate sqlSessionTemplate) {
        this.sqlSessionTemplate = sqlSessionTemplate;
    }

}
```





2.编写ApplicationContext.xml文件

将spring-dao.xml集成到总的ApplicationContext.xml中

```xml
    <import resource="spring-dao.xml" />
```

并在ApplicationContext.xml中注册UserMapperImpl对象

```xml
    <bean id="UserMapperImpl" class="com.nidie.mapper.UserMapperImpl">
        <property name="sqlSessionTemplate" ref="sqlSession" />
    </bean>
```

spring-dao.xml文件可以固定,不用修改太多的东西

当学springMVC时也可以写一个Spring-mvc.xml文件,专注于编写MVC部分,最后集成到ApplicationContext.xml中,这样比较分工明确



3.测试

```java
    @Test
    public void testQueryUsers() {
        ApplicationContext context = new ClassPathXmlApplicationContext("ApplicationContext.xml");
        UserMapper userMapperImpl = context.getBean("UserMapperImpl", UserMapper.class);
        for (User queryUser : userMapperImpl.queryUsers()) {
            System.out.println(queryUser);
        }
    }
```

摒弃了Mybatis的配置,直接将Mybatis配置集成在Spring中





##### Spring整合Mybatis的方法二

1.创建pojo实体类User

```java
public class User {

    private int id;

    private String name;

    private String pwd;

    public User() {
    }

    public User(int id, String name, String pwd) {
        this.id = id;
        this.name = name;
        this.pwd = pwd;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPwd() {
        return pwd;
    }

    public void setPwd(String pwd) {
        this.pwd = pwd;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", pwd='" + pwd + '\'' +
                '}';
    }

}
```



2.编写接口UserMapper

```java
public interface UserMapper {

    List<User> queryUsers();

}
```



3.创建UserMapper的实现类UserMapperImpl并继承SqlSessionDaoSupport

```java
public class UserMapperImpl2 extends SqlSessionDaoSupport implements UserMapper {

    public List<User> queryUsers() {
        return getSqlSession().getMapper(UserMapper.class).queryUsers();
    }

}
```



4.将UserMapperImpl注入到Spring中,并给父类SqlSessionDaoSupport的属性SqlSessionFactory一个值

```xml
    <bean id="userMapperImpl2" class="com.nidie.mapper.UserMapperImpl2" >
        <property name="sqlSessionFactory" ref="sqlSessionFactory" />
    </bean>
```



5.测试

```java
    @Test
    public void testQueryUsers() {
        ApplicationContext context = new ClassPathXmlApplicationContext("ApplicationContext.xml");
        UserMapper userMapperImpl = context.getBean("userMapperImpl2", UserMapper.class);
        for (User queryUser : userMapperImpl.queryUsers()) {
            System.out.println(queryUser);
        }
    }
```





### 12.声明式事务

#### 回顾事务

- 把一组业务当成一个业务来做,要么都成功,要么都失败
- 事务在项目开发中,十分的重要,涉及到数据的一致性问题,不能马虎
- 确保完整性和一致性



==事务的ACID原则:==

- A: 原子性
- C: 一致性
- I: 隔离性
  - 多个业务可能操作同一个资源,防止数据损坏
- D: 持久性
  - 事务一旦提交,无论系统发生什么问题,结果都不会再被影响,被持久化地写到存储器中



实例:

1.编写ApplicationContext.xml spring-dao mybatis-config.xml 配置文件

2.编写pojo实体类User

3.编写UserMapper接口类

```java
public interface UserMapper {

    List<User> queryUsers();

//    添加一个用户
    int addUser(User user);

//    删除一个用户
    int deleteUser(@Param("id") int id);

}
```

4.编写UserMapper.xml配置,并制造错误,将delete语句写错

```xml
<?xml version = "1.0" encoding = "UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.nidie.mapper.UserMapper">

    <select id="queryUsers" resultType="user" >
        select * from mybatis.user;
    </select>

    <insert id="addUser" parameterType="User">
        insert into mybatis.user(id , name , pwd) values(#{id} , #{name} , #{pwd});
    </insert>

    <delete id="deleteUser" parameterType="Integer" >
        deletes from mybatis.user where id = #{id}
    </delete>

</mapper>
```

5.编写UserMapperImpl实体类,并将addUser()和deleteUser()写到queryUsers()方法中

```java
public class UserMapperImpl extends SqlSessionDaoSupport implements UserMapper {

    public List<User> queryUsers() {
        UserMapper mapper = getSqlSession().getMapper(UserMapper.class);
        User user = new User(4, "你爹", "nidnia");
        mapper.addUser(user);
        mapper.deleteUser(4);

        return mapper.queryUsers();
    }

    public int addUser(User user) {
        return getSqlSession().getMapper(UserMapper.class).addUser(user);
    }

    public int deleteUser(int id) {
        return getSqlSession().getMapper(UserMapper.class).deleteUser(3);
    }

}
```

6.测试

```java
@Test
public void test1(){
    ApplicationContext context = new ClassPathXmlApplicationContext("ApplicationContext.xml");
    UserMapper userMapperImpl = context.getBean("userMapperImpl", UserMapper.class);

    List<User> users = userMapperImpl.queryUsers();
    for (User user : users) {
        System.out.println(user);
    }
}
```

结果

```java
org.springframework.jdbc.BadSqlGrammarException: 
### Error updating database.  Cause: com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'deletes from mybatis.user where id = 4' at line 1
### The error may exist in class path resource [com/nidie/mapper/UserMapper.xml]
### The error may involve com.nidie.mapper.UserMapper.deleteUser-Inline
### The error occurred while setting parameters
### SQL: deletes from mybatis.user where id = ?
### Cause: com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'deletes from mybatis.user where id = 4' at line 1
; bad SQL grammar []; nested exception is com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'deletes from mybatis.user where id = 4' at line 1

	at org.springframework.jdbc.support.SQLErrorCodeSQLExceptionTranslator.doTranslate(SQLErrorCodeSQLExceptionTranslator.java:234)
	at org.springframework.jdbc.support.AbstractFallbackSQLExceptionTranslator.translate(AbstractFallbackSQLExceptionTranslator.java:72)
	at org.mybatis.spring.MyBatisExceptionTranslator.translateExceptionIfPossible(MyBatisExceptionTranslator.java:74)
	at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:440)
	at com.sun.proxy.$Proxy7.delete(Unknown Source)
	at org.mybatis.spring.SqlSessionTemplate.delete(SqlSessionTemplate.java:303)
	at org.apache.ibatis.binding.MapperMethod.execute(MapperMethod.java:72)
	at org.apache.ibatis.binding.MapperProxy.invoke(MapperProxy.java:57)
	at com.sun.proxy.$Proxy8.deleteUser(Unknown Source)
	at com.nidie.mapper.UserMapperImpl.queryUsers(UserMapperImpl.java:17)
	at MyTest.test1(MyTest.java:16)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:50)
	at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
	at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:47)
	at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
	at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:325)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:78)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:57)
	at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
	at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
	at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:68)
	at com.intellij.rt.execution.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:47)
	at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:242)
	at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:70)
Caused by: com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'deletes from mybatis.user where id = 4' at line 1
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at com.mysql.jdbc.Util.handleNewInstance(Util.java:425)
	at com.mysql.jdbc.Util.getInstance(Util.java:408)
	at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:944)
	at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3978)
	at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3914)
	at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:2530)
	at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2683)
	at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2495)
	at com.mysql.jdbc.PreparedStatement.executeInternal(PreparedStatement.java:1903)
	at com.mysql.jdbc.PreparedStatement.execute(PreparedStatement.java:1242)
	at org.apache.ibatis.executor.statement.PreparedStatementHandler.update(PreparedStatementHandler.java:47)
	at org.apache.ibatis.executor.statement.RoutingStatementHandler.update(RoutingStatementHandler.java:74)
	at org.apache.ibatis.executor.SimpleExecutor.doUpdate(SimpleExecutor.java:50)
	at org.apache.ibatis.executor.BaseExecutor.update(BaseExecutor.java:117)
	at org.apache.ibatis.executor.CachingExecutor.update(CachingExecutor.java:76)
	at org.apache.ibatis.session.defaults.DefaultSqlSession.update(DefaultSqlSession.java:197)
	at org.apache.ibatis.session.defaults.DefaultSqlSession.delete(DefaultSqlSession.java:212)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:426)
	... 29 more
```

虽然SQL语句错误,但是queryUsers()和addUser()方法依然成功执行,不符合事务的一错都错





#### spring中的事务管理

- 声明式事务: AOP的运用
- 编程式事务: 需要在代码中,进行事务的管理



思考:

为什么需要事务?

- 如果不配置事务,可能存在数据提交不一致的情况
- 如果不在Spring中去配置声明式事务,我们就需要在代码中手动配置事务
- 事务在项目的开发中十分重要,涉及到数据的一致性和完整性问题,不容马虎



##### 声明式事务

配置spring-dao.xml

```xml
    <!--结合AOP实现事务的织入-->
    <!--配置事务通知:-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <!--给哪些方法配置事务-->
        <!--配置事务的传播特性: propagation REQUIRED:默认选择-->
        <tx:attributes>
            <tx:method name="add" propagation="REQUIRED" />
            <tx:method name="delete" propagation="REQUIRED" />
            <tx:method name="update" propagation="REQUIRED" />
            <tx:method name="query" read-only="true" />
            <tx:method name="*" propagation="REQUIRED" />
        </tx:attributes>
    </tx:advice>

    <!--配置事务切入-->
    <aop:config>
        <aop:pointcut id="txPointCut" expression="execution(* com.nidie.mapper.*.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointCut" />
    </aop:config>
```

先配置事务通知,需要修改数据的方法可以设置传播特性为REQUIRED

没有涉及到修改数据的方法,则可以改成read-only="true",这个方法就只读取数据库中的数据,而不进行任何修改

然后配置切入点

将事务配置到切入点中

运行后只要任何一个方法有错误,就不会被提交,数据库的数据就不会被修改,保证了数据的安全性





## SpringMVC

ssm: mybatis + Spring + SpringMVC

**MVC三层架构:**



JavaSE: 认真学习,老师带,入门快

JavaWeb: 认真学习,老师带,入门快

SSM框架: 研究官方文档,锻炼自学能力,锻炼项目能力



SpringMVC + Vuew + SpringBoot + SpringCloud + Linux



Spring: IOC 和 AOP



SpringMVC: SpringMVC的执行流程

SpringMVC: SSM框架整合



MVC: Model(模型:dao , service)	View(视图:JSP)	Controller(控制层:Servlet)

- MVC是模型(Model),视图(View),控制器(Controller)的简写,是一种软件设计规范
- 是将业务逻辑,数据,显示分离的方法来组织代码
- **MVC主要作用是降低了视图与业务逻辑见的双向耦合**
- MVC不是一种设计模式,**MVC是一种架构模式**



SpringMVC的特点:

1. 轻量级,简单易学
2. 搞笑,基于请求响应的MVC框架
3. 与Spring兼容性好,无缝结合
4. 约定优于配置
5. 功能强大: RESTful,数据验证,格式化,本地化,主题等
6. 简介灵活



Spring的web框架围绕DispatcherServlet[调度Servlet]设计



- dao
- service
- servlet
  - 转发
  - 重定向
- jsp/html



**JSP:本质就是一个Servlet**



假设:你的项目的架构,是设计好的,还是演进的?



Java更适用于大型项目,大型项目比较维护方便



![1629539719(1)](E:\files\JAVA\笔记\2021\resources\SpringMVC\1629539719(1).png)





SpringMVC原理:

​	当发起请求时被前置的控制器拦截到请求,根据请求参数生成代理请求,找到请求对应的实际控制器,控制器处理请求,创建数据模型,访问数据库,将模型响应给中心控制器,控制器使用模型与视图渲染视图结果,将结果返回给中心控制器,再将结果返回给请求者

![1629539987(1)](E:\files\JAVA\笔记\2021\resources\SpringMVC\1629539987(1).png)





### 回顾Servlet

创建项目

1.创建一个普通maven项目,删除src文件夹,再创建一个新的module,依旧是普通的maven项目

2.导包

```xml
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.2</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
    </dependencies>
```

3.右键项目add Framework Support

4.勾选Web Application,选择版本,点击确定

5.导入module的依赖

```xml
<dependency>
    <groupId>
</dependency>
```

6.在java下创建com.xxx.servlet包

7.编写HelloServlet类

```java
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1.获取前端参数
        String method = req.getParameter("method");
        if("method".equals("add")){
            req.getSession().setAttribute("msg" , "执行了add方法");
        }
        if ("method".equals("delete")) {
            req.getSession().setAttribute("msg" , "执行了delete方法");
        }

        //2.调用业务层


        //3.视图转化或者重定向
        req.getRequestDispatcher("/WEB-INF/jsp/test.jsp").forward(req , resp);

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req , resp);
    }

}
```

8.编写接收的页面test.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

${msg}

</body>
</html>
```

9.将servlet注册到web.xml文件中

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <servlet>
        <servlet-name>hello</servlet-name>
        <servlet-class>com.nidie.servlet.HelloServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>hello</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>

    <!--session超时设置-->
<!--    <session-config>-->
<!--        <session-timeout>15</session-timeout>-->
<!--    </session-config>-->

    <!--欢迎页面-->
    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>

</web-app>
```

10.配置tomcat

注意点:将Deployment的Application context修改为自己想要的路径

11.测试





### HelloSpringMVC



#### 第一个SpringMVC程序HelloSpringMVC

步骤:

1.创建普通Maven模块,并将其设置为Web Application

 2.编写web.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!--1.注册DispatcherServlet-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--关联一个springmvc的配置文件:【servlet-name】-servlet.xml-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc-servlet.xml</param-value>
        </init-param>
        <!--启动级别-1-->
        <load-on-startup>1</load-on-startup>
    </servlet>


    <!--/ 匹配所有的请求；（不包括.jsp）-->
    <!--/* 匹配所有的请求；（包括.jsp）-->
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>
```

param-value表示寻找springmvc-servlet.xml配置文件

servlet-mapping标签下的url-pattern标签表示拦截哪个路径的请求



3.创建controller包,并编写HelloController类

```java
public class HelloController implements Controller {

    /**
     * Process the request and return a ModelAndView object which the DispatcherServlet
     * will render. A {@code null} return value is not an error: it indicates that
     * this object completed request processing itself and that there is therefore no
     * ModelAndView to render.
     *
     * @param request  current HTTP request
     * @param response current HTTP response
     * @return a ModelAndView to render, or {@code null} if handled directly
     * @throws Exception in case of errors
     */
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        //ModelAndView 模型和视图
        ModelAndView mv = new ModelAndView();

        //封装对象,放在ModelAndView中.Model
        //ModelAndView存数据
        mv.addObject("msg" , "HelloSpringMVC!");

        //封装要跳转的视图,放在ModelAndView中
        //设置要跳转到的地方
        mv.setViewName("hello");
        // : /WEB-INF/jsp/hello.jsp

        return mv;
    }

}
```

将msg参数保存在mv中

并转发到hello.jsp文件中



4.编写springmvc-servlet.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>

    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>

    <!--视图解析器:DispatcherServlet给他的ModelAndView-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="InternalResourceViewResolver">
        <!--前缀-->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <!--后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--Handler-->
    <bean id="/hello" class="com.nidie.controller.HelloController" />

</beans>
```

前缀和后缀表示mv要跳转的文件的位置

即:prefix + 文件名 + .jsp

mv.setViewName("hello")就编程mv.setViewName("/WEB-INF/jsp/hello.jsp");



5.在WEB-INF文件夹下创建jsp文件夹并创建hello.jsp接收msg

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

${msg}

</body>
</html>
```

6.运行并请求http://localhost:8080/hello

7.返回结果





注意: 如果项目没有运行,出现404错误,看看Tomcat中你这个项目Deployment下有没有lib依赖文件夹,如果没有的话,则创建lib文件夹,并把包导入到lib文件夹中,再重启运行



```xml
在SpringMVC中, /和/*的区别
 /: 只会匹配所有的请求,不会匹配jsp
 /*: 匹配所有的请求,包括jsp
```

```
SpringMVC三要素,没有的话SpringMVC就不能运行
<!--处理器映射器-->


<!--处理器适配器-->


<!--视图解析器-->
```



### 深入SpringMVC



MVVM: M V VM(ViewModel:双向绑定功能 前后端分离的核心)



Spring:大杂烩,我们可以将SpringMVC中所有要用到的Bean,注册到Spring中



#### SpringMVC运行流程:

虚线部分是我们自己编写代码,实现部分为SpringMVC为我们处理

![1629546192(1)](E:\files\JAVA\笔记\2021\resources\SpringMVC\1629546192(1).png)

#### 简要分析执行流程

1. DispatcherServlet表示前置控制器,是整个SpringMVC的控制中心.用户发出请求,DispatcherServlet接收请求并拦截请求
   - 我们架设请求的url为: http://localhost:8080/SpringMVC/hello
   - 如上url拆分成三部分:
     - http://localhost:8080服务器域名
     - SpringMVC部署在服务器上的web站点
     - hello表示控制器
     - 通过分析,如上url表示为:请求位于服务器localhost:8080上的SpringMVC站点的hello控制器
2. HandlerMapping为处理器映射.DispatcherServlet调用
3. HandlerMapping,HandlerMapping根据请求url查找Handler
4. HandlerExecution表示具体的Handler,其主要作用是根据url查找控制器,如上url被查找控制器为:hello
5. HandlerExecution将解析后的信息传递给DispatcherServlet,如解析控制器映射等
6. HandlerAdapter表示处理器适配器,其按照特定的规则去执行Handler
7. Handler让具体的Controller执行
8. HandlerAdapter将视图逻辑名或者模型传递给DispatcherServlet
9. DispatcherServlet调用视图解析器(ViewResolver)来解析HandlerAdapter传递的逻辑视图名
10. 视图解析器将解析的逻辑视图名传给DispatcherServlet
11. DispatcherServlet根据视图解析器解析的视图结果,调用具体的视图
12. 最终视图呈现给用户



细分:
第一步: 1-4步是适配请求的作用

第二步: 5-8步请求的具体作用,并返回结果

第三步: 视图解析



### 注解开发SpringMVC

1.配置springmvc-servlet.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--自动扫描包,让制定包下的注解生效,由IOC容器统一管理-->
    <context:component-scan base-package="com.nidie.controller" />

    <!--让SpringMVC不处理静态资源 .css .js .html .mp3 .mp4-->
    <mvc:default-servlet-handler />

    <!--
    支持MVC注解驱动
        在spring中一般采用@RequestMapping注解来完成映射关系
        要想使@RequestMapping注解生效
        必须向上下文中注册DefaultAnnotationHandlerMapping
        和一个AnnotationMethodHandlerAdapter实例
        这两个实例分别在类级别和方法级别处理
        而annotation-driven配置帮助我们自动完成上述两个实例的注入
        -->
    <mvc:annotation-driven />

    <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"
          id="internalResourceViewResolver">
        <!--前缀-->
        <property name="prefix" value="/WEB-INF/jsp/" />
        <!--后缀-->
        <property name="suffix" value=".jsp" />
    </bean>

</beans>
```



2.web.xml

```xml
    <!--1.注册DispatcherServlet-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--关联一个springmvc的配置文件:【servlet-name】-servlet.xml-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc-servlet.xml</param-value>
        </init-param>
        <!--启动级别-1-->
        <load-on-startup>1</load-on-startup>
    </servlet>


    <!--/ 匹配所有的请求；（不包括.jsp）-->
    <!--/* 匹配所有的请求；（包括.jsp）-->
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```



完整实现步骤:

1. 新建一个web项目
2. 导入相关jar包
3. 编写web.xml,注册DispatcherServlet
4. 编写springmvc配置文件
5. 接下来就是去创建对应的控制类,controller
6. 最后完善前端视图和controller之间的对应
7. 在Tomcat中部署项目,添加lib文件夹,导入jar包
8. 测试运行调试



使用SpringMVC必须配置的三大件:

1. **处理器映射器**
2. **处理器适配器**
3. **视图解析器**

通常,我们只需要手动配置视图解析器,而处理器映射器和处理器适配器只需要开启注解驱动即可,而省去大段的xml配置





### Controller及RestFul风格

控制器Controller

- 控制器负责提供访问应用程序的行为,通常通过接口定义或注解定义两种方法实现
- 控制器负责解析用户的请求并将其转换为一个模型
- 在SpringMVC中一个控制器类可以包含多个方法
- 在SpringMVC中,对于Controller的配置方式有很多种





```java
@FunctionalInterface
//实现该接口的类获得控制器功能
public interface Controller {

   /**
    * Process the request and return a ModelAndView object which the DispatcherServlet
    * will render. A {@code null} return value is not an error: it indicates that
    * this object completed request processing itself and that there is therefore no
    * ModelAndView to render.
    * @param request current HTTP request
    * @param response current HTTP response
    * @return a ModelAndView to render, or {@code null} if handled directly
    * @throws Exception in case of errors
    */
   //处理请求并且返回一个模型和视图对象
   @Nullable
   ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception;

}
```



#### 使用Controller

##### 方式一:直接写一个Controller类实现Controller接口

1.创建Controller类,实现Controller接口,然后在handleRequest方法中写业务

```java
package com.nidie.controller;

import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * 只要实现了Controller接口的类,说明这就是一个控制器了
 * @author nidie
 */
public class ControllerTest1 implements Controller {

    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        ModelAndView mv = new ModelAndView();
        //封装数据
        mv.addObject("msg" , "ControllerTest1");
        //设置跳转
        mv.setViewName("test");

        return mv;
    }

}
```

ModelAndView mv = new ModelAndView();

创建一个模型和视图对象

然后在mv中封转数据

再设置跳转

2.编写web.xml类

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    
    <!--1.配置DispatcherServlet-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>
```

配置DispatcherServlet包

然后绑定springmvc-servlet.xml配置文件

load-on-startup:

1. load-on-startup 元素标记容器是否应该在web应用程序启动的时候就加载这个servlet，(实例化并调用其init()方法)。
2. 它的值必须是一个整数，表示servlet被加载的先后顺序。
3. 如果该元素的值为负数或者没有设置，则容器会当Servlet被请求时再加载。
4. 如果值为正整数或者0时，表示容器在应用启动时就加载并初始化这个servlet，值越小，servlet的优先级越高，就越先被加载。值相同时，容器就会自己选择顺序来加载。

设置load-on-startup为1

然后绑定url路径



3.编写springmvc-servlet.xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd">

<!--    &lt;!&ndash;自动扫描包,让制定包下的注解生效,由IOC容器统一管理&ndash;&gt;-->
<!--    <context:component-scan base-package="com.nidie.controller" />-->

<!--    &lt;!&ndash;让SpringMVC不处理静态资源 .css .js .html .mp3 .mp4&ndash;&gt;-->
<!--    <mvc:default-servlet-handler />-->

<!--    &lt;!&ndash;-->
<!--    支持MVC注解驱动-->
<!--        在spring中一般采用@RequestMapping注解来完成映射关系-->
<!--        要想使@RequestMapping注解生效-->
<!--        必须向上下文中注册DefaultAnnotationHandlerMapping-->
<!--        和一个AnnotationMethodHandlerAdapter实例-->
<!--        这两个实例分别在类级别和方法级别处理-->
<!--        而annotation-driven配置帮助我们自动完成上述两个实例的注入-->
<!--        &ndash;&gt;-->
<!--    <mvc:annotation-driven />-->

    <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"
          id="internalResourceViewResolver">
        <!--前缀-->
        <property name="prefix" value="/WEB-INF/jsp/" />
        <!--后缀-->
        <property name="suffix" value=".jsp" />
    </bean>

    <bean name="/t1" class="com.nidie.controller.ControllerTest1" />

</beans>
```

mvc的配置文件只留下视图解析器

<bean name="/t1" class="com.nidie.controller.ControllerTest1" />

name即是url路径的后缀



4.在WEB-INF下创建jsp类,然后创建test.jsp页面文件,并接收来自Controller的msg

```jsp
<%--
  Created by IntelliJ IDEA.
  User: nidie
  Date: 2021/8/22
  Time: 17:42
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    ${msg}
</body>
</html>
```



5.配置Tomcat的Deployment,将项目发布,并且配置lib文件夹,在lib文件夹中导入所有的包



6.运行测试,url路径: http://localhost:8080/t1



注意点:

1. 虽然没有显示地调用处理器映射器和处理器适配器,但是项目依然可以运行,因为Spring已经隐式地调用了
2. 在工作实战中,<bean name="/t1" class="com.nidie.controller.ControllerTest1" />都不用配置



说明:

- 实现结构Controller定义控制器是较老的方法
- 缺点:一个控制器中只有一个方法,如果需要多个方法则需要定义多个Controller;定义的方式比较麻烦





##### 方式二:使用注解@Controller

```xml
@Component	组件
@Service	service
@Controller	controller
@Repository	dao
```



- @Controller注解类型用于声明Spring类的实例是一个控制器
- Spring可以使用扫描机制来找到应用程序中所有基于注解的控制类,为了保证Spring能找到你的控制器,需要在配置文件中声明组件扫描



第一步: 在配置文件中声明组件扫描

```xml
<!-- 自动扫描指定的包,下面所有注解类交给IOC容器管理 -->
<context:component-scan base-package="com.nidie.controller" />
```



第二步:编写Controller类,然后用@Controller注解,在其中写方法

```java
package com.nidie.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * 方式二:使用注解
 * {@Controller}代表这个类会被Spring接管,被这个注解注解的类中的所有方法如果返回值是String
 * 并且由具体页面可以跳转,那么就会被视图解析器解析
 * @author nidie
 */
@Controller
public class ControllerTest2 {

    @RequestMapping("/t2")
    public String test1(Model model){
        model.addAttribute("msg" , "ControllerTest2");

        return "test";
    }

    @RequestMapping("/t3")
    public String test2(Model model){
        model.addAttribute("msg" , "ControllerTest3");

        return "test";
    }

}
```

@Controller代表这个类会被Spring接管,被这个注解注解的类中的所有方法如果返回值是String并且由具体页面可以跳转,那么就会被视图解析器解析



可以发现,我们两个请求都可以指向同一个视图,但是页面的结果是不一样的,从这里可以看出视图是可以被复用的,而控制器和视图之间是弱耦合关系



### RequestMapping说明

@RequestMapping

- @RequestMapping注解用于映射url到控制器类或一个特定的处理程序方法.可用于类或方法上.用于类上,表示类中的所有响应请求的方法都是以该地址作为父路径
- 为了测试结论更加准确,我们可以加上一个项目名测试myweb
- 只注解在方法上面

```java
    @RequestMapping("/t1")
    public String test1(Model model){
        model.addAttribute("msg" , "ControllerTest1");

        return "test";
    }
```

访问路径: http://localhost:8080/项目名/h1

- 同时注解类和方法

```java
@RequestMapping("/modoe")
public class ControllerTest2 {

    @RequestMapping("/t1")
    public String test1(Model model){
        model.addAttribute("msg" , "ControllerTest1");

        return "test";
    }
```

路径: http://localhost:8080/modoe/h1



##### RestFul风格

​	RestFul就是一个资源定位及资源操作的风格.不是标准也不是协议,只是一种风格.基于这个风格设计的软件可以更简洁,更有层次,更易于实现缓存等机制



功能

- 资源: 互联网所有的事务都可以被抽象为资源
- 资源操作: 使用POST,DELETE,PUT,GET,使用不同方法对资源进行操作
- 分别对应 添加,删除,修改,查询



传统方式操作资源: 通过不同的参数来实现不同的效果!方法单一, POST和GET

- http://127.0.0.1/item/queryItem.action?id=1 查询,GET

- http://127.0.0.1/item/saveItem.action 新增,POST

- http://127.0.0.1/item/updateItem.action 更新,POST

- http://127.0.0.1/item/deleteItem.action?id=1 删除,GET或POST



使用RESTful操作资源:可以通过不同的请求方式来实现不同的效果,如下:请求地址一样,但是功能可以不同

- http://127.0.0.1/item/1 查询,GET
- http://127.0.0.1/item 新增,POST
- http://127.0.0.1/item 更新,PUT
- http://127.0.0.1/item/1 删除,DELETE

  

RestFul风格

```java
package com.nidie.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

/**
 * @author nidie
 */
@Controller
public class RestFulController {

    //原来的方式: http://localhost:8080/add?a=1&b=2
    //RestFul: http://localhost:8080/add/a/b

    //@RequestMapping(path = "/add/{a}/{b}" , method = RequestMethod.GET)
    @PostMapping("/add/{a}/{b}")
    public String test1(@PathVariable int a , @PathVariable String b , Model model){
        String result = a + b;
        model.addAttribute("msg" , "结果1为:" + result);

        return "test";
    }

    @PostMapping("/add/{a}/{b}")
    public String test2(@PathVariable int a , @PathVariable String b , Model model){
        String result = a + b;
        model.addAttribute("msg" , "结果2为:" + result);

        return "test";
    }

}
```



**使用method属性指定请求类型**

​	用于约束请求的类型,可以收窄请求范围.指定请求谓词的类型如GET,POST,HEAR,OPTIONS,PUT,PATCH,DELETE,TRACE等



所有的地址栏请求默认都会是HTTP GET类型的

方法级别的注解变体有如下几个: 组合注解



使用路径变量的好处

- 使路径变得更加简洁
- 获得参数更加方便,框架会自动进行类型转换
- 通过路径变量的类型可以约束访问路径,如果类型不一样,则访问不到对应的请求方法,如这里访问时的路径是/commit/1/a,则路径与方法不匹配,而不会是参数转换失败



### 结果跳转方法

#### ModelAndView

​	设置ModelAndView对象,根据view的名称,和视图解析器跳到指定的页面

​	页面: {视图解析器前缀} + viewName + {视图解析器后缀}

```xml
    <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"
          id="internalResourceViewResolver">
        <!--前缀-->
        <property name="prefix" value="/WEB-INF/jsp/" />
        <!--后缀-->
        <property name="suffix" value=".jsp" />
    </bean>
```

对应的controller类

```java
/**
 * 只要实现了Controller接口的类,说明这就是一个控制器了
 * @author nidie
 */
public class ControllerTest1 implements Controller {

    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        ModelAndView mv = new ModelAndView();
        //封装数据
        mv.addObject("msg" , "ControllerTest1");
        //设置跳转
        mv.setViewName("test");

        return mv;
    }

}
```



#### ServletAPI

​	通过设置ServletAPI,不需要视图解析器

1. 通过HttpServletResponse进行输出
2. 通过HttpServletResponse实现重定向
3. 通过HttpServletResponse实现转发

```java
@Controller
public class ResultGo{
    
    @RequestMapping("/result/t1")
    public void test1(HttpServletRequest req , HttpServletResponse rsp) throws IOException
    {
        rsp.getWritter().println("你爹1");
    }
    
    @RequestMapping("/result/t2")
    public void test2(HttpServletRequest req , HttpServletResponse rsp) throws IOException
    {
        rsp.sendRedirect("/index.jsp");
    }
    
    @RequestMapping("/result/t3")
    public void test3(HttpServletRequest req , HttpServletResponse rsp) throws IOException
    {
        req.setAttribute("msg" , "/result/t3");\
        req.getRequestDispatcher("/WEB-INF/jsp/test.jsp").forward(req , rsp);
    }
    
}
```



#### SpringMVC

要先关闭视图解析器



通过SpringMVC来实现转发和重定向 无需视图解释器

```java
@Controller
public class ResultSpringMVC{
    
    @RequestMapping("/rsm/t1")
    public String test1(){
        //转发
        return "/index.jsp";
    }
    
    @RequestMapping("/rsm/t2")
    public String test2(){
        //转发二
        return "forward:/index.jsp";
    }
    
    @RequestMapping("rsm/t3")
    public String test3(){
        //重定向
        return "redirect:/index.jsp";
    }
    
}
```





### 数据处理

#### 处理提交数据

##### 1.提交的域名称和处理方法的参数名一致



提交数据: http://localhost:8080/hello?name=kuangshen



处理方法:

```java
@RequestMapping("/hello")
public String hello(String name){
    System.out.println(name);
    return "hello";
}
```

后台输出: kuangshen



##### 2.提交的域名称和处理方法的参数名不一致

提交数据: http://localhost:8080/hello?username=kuangshen

处理方法

```java
//@RequestParam("username")
@RequestMapping("/hello")
public String hello(@RequestParam("username") String name){
    System.out.println(name);
    return "hello";
}
```



##### 3.提交的是一个对象

要求提交的表单域和对象的属性名一致,参数使用对象即可





#### 数据显示到前端

##### 第一种:通过ModelAndView 



##### 第二种:通过ModelMap



##### 第三种:通过Model

Model

```java
@RequestMapping("/ct2/hello")
public String hello(@RequestParam("username") String name , Model model){
    //封装要显示到视图中的数据
    //相当于req.setAttribute("name" , name);
    model.addAttribute("msg" , name);
    System.out.println(name);
    return "test";
}
```



##### 对比

就对于新手而言简单来说使用区别就是:

```java
Model 只有寥寥几个方法只适合用于储存数据,简化了新手对于Model对象的操作和理解;

ModelMap 继承了LinkedMap,除了实现自身的一些方法,同样的继承LinkedMap的方法和特性;

ModelAndView 可以在储存数据的同时,可以进行设置返回的逻辑视图,进行控制现实层的跳转;
```



```java
LinkedHashMap
    
Modelmap: 继承了LinkedHashMap,所以他拥有LinkedHashMap的全部功能
    
Model: 精简版
```





### 乱码问题

测试步骤:

1.我们可以在首页编写一个提交的表单

```jsp
<form action="/e/t" method="post">
    <input value="text" name="name" />
    <input type="submit" />
</form>
```



2.后台编写对应的处理类

```java
@Controller
public class Encoding{
    
    @RequestMapping("/e/t")
    public String test(Model model , String name){
        model.addAttribute("msg" , name);//获取表单提交的值
        return "test"; //跳转到test页面显示输入的值
    }
    
}
```



3.输入中文测试,发现乱码





### JSON

前后端分离时代:
后端部署后端,提供接口,提供数据;



前端独立部署,辅助渲染后端的数据



- JSON(JavaScript Object Notation,JS对象标记)是一种轻量级的数据交换格式,目前使用特别广泛
- 采用完全独立于编程语言的文本格式来存储和表示数据
- 简介和清晰的层次结构使得JSON成为理想的数据交换功能
- 易于人阅读和编写,同时也易于机器解析和生成,并有效地提升网络传输效率



在JavaScript语言中,一切都是对象.因此,任何JavaScript支持的类型都可以通过JSON来表示,例如字符串,数字,对象,数组等.语法格式:

- 对象表示为键值对,数据由逗号分隔
- 花括号保存对象
- 方括号保存数组

```json
{"name":"你爹"}
{"age":"3"}
{"sex":"男"}
```

- JSON是JavaScript对象的字符串表示法,它使用文本表示一个JS对象的信息,本质是一个字符串


```json
var obj = {'a':'Hello',b:"Word"};//这是一个对象,键名也是可以使用引号包裹的
var json = '{"a":"hello","b":"World"}';//这是一个JSON字符串,本质是一个字符串
```



JSON和JavaScript对象互转

- 要实现从JSON字符串转换为JavaScript对象,要使用JSON.parese()方法

  ```javascript
  var obj = JSON.parse('{"a":"Hello","b":"World"}');
  //结果是{a:'Hello' , b:'World'}
  ```

- 要实现从JavaScript对象转换为JSON字符串,使用JSON.stringify()方法

```javascript
var json = JSON.stringify({a:'Hello',b:'World'});
//结果是'{"a":"Hello" , "b":"World"}'
```



#### Jackson

Controller生成Json对象

- Jackson应该是目前比较好的json解析工具了
- 其他的json解析工具还有阿里巴巴的fastjson等
- 使用Jackson,需要导入它的jar包

```xml
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.10.0</version>
</dependency>
```



1.导包

2.编写web.xml和springmvc-serlvet.xml配置文件

3.编写Pojo实体类

4.编写Controller类

```java
    /**
     * {@ResponseBody} 方法就不会走视图解析器,会直接返回一个字符串
     * @return String 要返回到的地址
     */
    @RequestMapping(value = "/j1" , produces = "application/json;charset=utf-8")
    @ResponseBody
    public String json1() throws JsonProcessingException {
        //Jackson中有一个对象ObjectMapper
        ObjectMapper mapper = new ObjectMapper();

        //创建一个对象
        User user = new User("你爹" , 20 , "男");

        return mapper.writeValueAsString(user);
    }
```

使用Jackson的ObjectMapper将java对象转换为JSON对象



5.配置tomcat项目

6.运行测试



#### 如果遇到乱码问题

1.原生态解决方式

通过@RequestMapping的produces属性来实现,修改下乱码

@RequestMapping(value = "/j1" , produces = "application/json;charset=utf-8")



2.乱码优化

乱码统一解决

我们可以在springmvc的配置文件上添加一段消息StringHttpMessageConverter转换配置

```xml
    <mvc:annotation-driven>
        <mvc:message-converters register-defaults="true">
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <constructor-arg value="UTF-8" />
            </bean>
            <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
                <property name="objectMapper">
                    <bean class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
                        <property name="failOnEmptyBeans" value="false" />
                    </bean>
                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>
```



```
@RestController 代表这个类下面只会返回字符串
```





返回一个List集合

```java
@RequestMapping(value = "/j2")
public String json2() throws JsonProcessingException {
    List<User> list = new ArrayList<User>();

    User user1 = new User("你爹" , 20 , "男");
    User user2 = new User("你爹" , 20 , "男");
    User user3 = new User("你爹" , 20 , "男");
    User user4 = new User("你爹" , 20 , "男");

    list.add(user1);
    list.add(user2);
    list.add(user3);
    list.add(user4);

    return new ObjectMapper().writeValueAsString(list);
}
```

结果

```json
[{"name":"你爹","age":20,"sex":"男"},{"name":"你爹","age":20,"sex":"男"},{"name":"你爹","age":20,"sex":"男"},{"name":"你爹","age":20,"sex":"男"}]
```



#### 格式化时间

Jackson ObjectMapper格式化时间为Timestamp,时间戳



1.使用纯Java SimpleDateFormat类来格式化时间

```java
    @RequestMapping(value = "/j3")
    public String json3() throws JsonProcessingException {
        return new ObjectMapper().writeValueAsString(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));
    }
```



2.使用ObjectMapper的configure方法

```java
    @RequestMapping(value = "/j3")
    public String json3() throws JsonProcessingException {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

        ObjectMapper mapper = new ObjectMapper();
        //使用ObjectMapper来格式化输出
        mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS , false);

        mapper.setDateFormat(sdf);

        //ObjectMapper,时间解析后的默认格式为:Timestamp,时间戳

        return new ObjectMapper().writeValueAsString(new Date());
    }
```

mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS , false); //将自动解析为Timestamp格式关闭

mapper.setDateFormat(sdf); //设置解析的时间格式





#### FastJson

​	fastjson.jar是阿里开发的一款专门用于Java开发的包,可以方便地实现json对象与JavaBean对象的转换,实现JavaBean对象与json字符串的转换,实现json对象与json字符串的转换.实现json的转换方法很多,最后的实现结果都是一样的



fastjson的pom依赖

```xml
<!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.60</version>
</dependency>
```



fastjson三个主要的类:

- JSONObject 代表json对象
  - JSONObject实现了Map接口,猜想JSONObject底层操作是由Map实现的	
  - JSONObject对象json对象,通过各种形式的get()方法可获取json对象中的数据,也可利用注入size(),isEmpty()等方法获取"键:值"对的个数和判断是否为空.其本质是通过实现Map接口并调用接口中的方法完成的
- JSONArray代表json对象数组
  - 内部是有List接口中的方法来完成操作
- JSON代表JSONObject和JSONArray的优化
- JSON类源码分析与使用
- 仔细观察这些方法,主要是实现json对象,json对象数组,javabean对象,json字符串之间的互相转化





### 整合SSM

#### 数据库环境

```mysql
CREATE DATABASE `ssmbuild`;

USE `ssmbuild`

DROP TABLE IF EXISTS `books`;

CREATE TABLE `books`(
    `bookID` INT(10) PRIMARY KEY AUTO_INCREMENT COMMENT '书id',
    `bookName` VARCHAR(100) NOT NULL COMMENT `书名`,
    `bookCounts` INT(11) NOT NULL COMMENT '数量',
    `detail` VARCHAR(200) NOT NULL COMMENT '描述',
    KEY `bookID` (`bookID`)
)ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO `books`(`bookName` , `bookCounts` , `detail`) VALUES('Java' , 1 , '从入门到放弃'),('MySql' , 10 , '从删库到跑路'),('Linux' , 5 , '从入门到入狱');
```



#### 基本环境搭建

1.新建一个Maven项目! ssmbuild , 添加web的支持

2.导入相关的pom依赖

```xml
<!--依赖: junit , 数据库驱动 , 连接池 , servlet , jsp ,
    mybatis , mybatis-spring , springmvc-->
<dependencies>
    <!--Junit-->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
    <!--数据库驱动-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.47</version>
    </dependency>
    <!-- 数据库连接池: c3p0 -->
    <dependency>
        <groupId>com.mchange</groupId>
        <artifactId>c3p0</artifactId>
        <version>0.9.5.2</version>
    </dependency>

    <!--Servlet - JSP -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>
        <version>2.2</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
    </dependency>

    <!--Mybatis-->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.2</version>
    </dependency>
    <!--Mybatis-Spring-->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>2.0.2</version>
    </dependency>

    <!--Spring-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.1.9.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>5.1.9.RELEASE</version>
    </dependency>
</dependencies>
```

3.在build中配置resources,来防止我们资源到处失败的问题

```xml
<build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
    </build>
```

4.配置spring-dao.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:contexe="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--1.关联数据库配置文件-->
    <contexe:property-placeholder location="classpath:db.properties" />


    <!--2.连接池
        dbcp: 半自动化操作,不能自动连接
        c3p0: 自动化操作(自动化地操作配置文件,并且可以自动设置到对象中)
        druid:
        hikari: SpringBoot 2.0以后默认的数据库连接池
    -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" >
        <property name="driverClass" value="${jdbc.driver}" />
        <property name="jdbcUrl" value="${jdbc.url}" />
        <property name="user" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />

        <!--c3p0连接池的私有属性-->
        <property name="maxPoolSize" value="30" />
        <property name="minPoolSize" value="10" />
        <!--关闭连接后不自动commit-->
        <property name="autoCommitOnClose" value="false" />
        <!--获取连接超时时间-->
        <property name="checkoutTimeout" value="10000" />
        <!--当获取连接失败重试次数-->
        <property name="acquireRetryAttempts" value="2" />
    </bean>

    <!--3.SqlSessionFactory-->

</beans>
```





















































































































