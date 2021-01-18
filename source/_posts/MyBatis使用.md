---
title: MyBatis使用
date: 2021-01-17 09:24:58
tags:
---

## MyBatis做了什么

+ 简化了模板代码的书写
+ 简化了数据库中数据映射为bean的过程ORM框架

使用JDBC需要步骤

+ 加载驱动

```
Class.forName("com.mysql.jdbc.Driver")
```

+ 向java.sql.DriverManager请求并获取Connection

```java
Connection con = DriverManager.getConnection(url , username , password );
```

+ 通过Connection获取PreparedStatement或Statement

```java
Statement stmt = con.createStatement();
PreparedStatement pstmt = con.prepareStatement(sql);
```

+ 执行sql

```java
ResultSet rs = stmt.executeQuery("SELECT * FROM ...");   
int rows = stmt.executeUpdate("INSERT INTO ...");    
boolean flag = stmt.execute(String sql);
```

+ 遍历结果

```java
while(rs.next()){    
  String name = rs.getString("name") ;    
  String pass = rs.getString(1) ; // 此方法比较高效    
}    
```

+ 关闭资源

```java
if(rs !=null){   // 关闭记录集    
  try {
    rs.close();
  } catch (SQLException e) {
    e.printStackTrace();
  }
}    
if(stmt !=null){   // 关闭声明    
  try {
    stmt.close();
  } catch (SQLException e) {
    e.printStackTrace();
  }
}
if(conn !=null){  // 关闭连接对象    
  try {
    conn.close();
  } catch (SQLException e) {
    e.printStackTrace();
  }
}
```

使用MyBatis的步骤

+ 构建 SqlSessionFactory

```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

+ SqlSessionFactory 中获取 SqlSession

```java
SqlSession session = sqlSessionFactory.openSession()
```

+ 使用SqlSession执行sql

```java
BlogMapper mapper = session.getMapper(BlogMapper.class);
Blog blog = mapper.selectBlog(101);
```

## MyBatis详细执行流程

`Resources`对象获取加载配置文件

`Configuration`通过加载构建配置类

`SqlSessionFactoryBuilder`builder模式构建`SqlSessionFactory`

`SqlSessionFactory`中含有`Configuration`配置类

通过`SqlSessionFactory`获取`SqlSession`类

`SqlSession`中含有`Executor`执行器和`Configuration`配置类

`Executor`中含有`Transaction`负责管理事务

执行CRUD

如果失败使用`Transaction`回滚

如果成功将数据映射

关闭资源

## MyBatis缓存

一级缓存：sqlsession级别的缓存。

二级缓存：mapper级别缓存，可以通过实现Cache来自定义缓存实现。

