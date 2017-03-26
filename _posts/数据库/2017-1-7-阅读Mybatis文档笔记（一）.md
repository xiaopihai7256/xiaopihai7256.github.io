---
layout: post
title: 阅读Mybatis文档笔记（一）
categories: 
 - 数据库
 - Mybatis
author: 杨比轩
---

> 笔记是在我使用过mybatis后再读文档的记录，不是入门教程。

## 一、基本原理

官方解释：

> 每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为中心的。SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder 获得。而 SqlSessionFactoryBuilder 则可以从 XML 配置文件或一个预先定制的 Configuration 的实例构建出 SqlSessionFactory 的实例。

1. 从mybatis.cfg.xml构建`SqlSessionFactoryBuilder `
2. 从`SqlSessionFactoryBuilder` 来`build()`出`SqlSessionFactor`
3. 使用`SqlSessionFactory`调用`openSession()`获取SqlSession对象
4. 最后就可以使用`SqlSession`加载`mapper.class`来执行XXXMappper.xml中的映射sql语句

```java
public class DBTools {

	//此类使用静态代码块对SqlSessionFactory实现单例初始化
	public static SqlSessionFactory sessionFactory;

	static{
		try{
			Reader reader = Resources.getResourceAsReader("mybatis.cfg.xml");

			sessionFactory = new SqlSessionFactoryBuilder().build(reader);
		}catch (Exception e) {
			e.printStackTrace();
		}
	}

	public static SqlSession getSession(){
		return sessionFactory.openSession();
	}
}
```

### mybatis配置

官方文档交代，构建SqlSessionFactory既可以使用java代码手动初始化，可以加载xml配置文件来实现，当然项目中都是使用后者。所以，如何配置xml就成了最基本的内容。

官方基本配置（添加了注释）：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

<!-- 配置mybatis运行环境 -->
  <environments default="development">
    <environment id="development">
    <!-- 事物管理 -->
    <!-- type="JDBC" 代表使用JDBC的提交和回滚来管理事务 -->
      <transactionManager type="JDBC"/>

      <!--连接池-->
      <!-- mybatis提供了3种数据源类型，分别是：POOLED,UNPOOLED,JNDI -->
			<!-- POOLED 表示支持JDBC数据源连接池 -->
			<!-- UNPOOLED 表示不支持数据源连接池 -->
			<!-- JNDI 表示支持外部数据源连接池 -->
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>

<!--映射方式 这里是添加单个的xml文件形式-->
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```

### SQL映射语句的执行

文章开始的demo中给出了获取SqlSession的方式，最后使用该对象就可以来执行mapper.xml文件中的sql语句里，具体的执行方式也有两种。

先看XXXmapper.xml是怎么写的，官方demo：
```xml
<?xml version="1.0" encoing="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.BlogMapper">
  <select id="selectBlog" resultType="Blog">
    select * from Blog where id = #{id}
  </select>
</mapper>
```
然后就是执行映射的sql语句的两种方式:

```java
//方式一
SqlSession session = sqlSessionFactory.openSession();
try {
  Blog blog = (Blog) session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);
} finally {
  session.close();
}

//方式二
SqlSession session = sqlSessionFactory.openSession();
try {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  Blog blog = mapper.selectBlog(101);
} finally {
  session.close();
}
```

一般来说，实际使用都是使用第二种方式，而且当框架整合完成的时候，mapper都是通过注入的方式来实现实例化，并不需要手动的初始化。而后就是使用mapper对象，来调用mapper接口的映射方法直接执行sql语句。

> 官方解释：第二种方法有很多优势，首先它不是基于字符串常量的，就会更安全；其次，如果你的 IDE 有代码补全功能，那么你可以在有了已映射 SQL 语句的基础之上利用这个功能。

到这里，还有一点，就是mapper.java的映射器，这个很简单，但是是必不可少的。

```java
public interface BlogMapper {

  Blog selectBlog(int id);
}
```

## 作用域和声明周期

1. 对象的声明周期和依赖注入框架

    依赖注入框架可以创建线程安全的、基于事务的 SqlSession 和映射器（mapper）并将它们直接注入到你的 bean 中，因此可以直接忽略它们的生命周期。

    > 也就是说，在spring等框架中，这是直接通过框架注入和管理的。

2. SqlSessionFactoryBuilder

    **这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了**。因此 SqlSessionFactoryBuilder 实例的最佳作用域是方法作用域（也就是局部方法变量）。你可以重用 SqlSessionFactoryBuilder 来创建多个 SqlSessionFactory 实例，但是最好还是不要让其一直存在以保证所有的 XML 解析资源开放给更重要的事情。

    > SqlSessionFactory被创建完成，就可以撇掉了，所以可以在方法或者代码段里申明局部的Builer来对Factory进行创建。

3. SqlSessionFactory

    **SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在**，没有任何理由对它进行清除或重建。使用 SqlSessionFactory 的最佳实践是在应用运行期间不要重复创建多次，多次重建 SqlSessionFactory 被视为一种代码“坏味道（bad smell）”。因此 SqlSessionFactory 的最佳作用域是应用作用域。有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式。

4. SqlSession

    **每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。**绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。也绝不能将 SqlSession 实例的引用放在任何类型的管理作用域中，比如 Serlvet 架构中的 HttpSession。

    > 线程不安全，最好放在方法作用域！

5. 映射器实例（Mapper Instances）

    作用域小于等于SqlSession作用域。最好还是放在方法作用域，
