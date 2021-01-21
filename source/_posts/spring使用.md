---
title: spring使用
date: 2021-01-18 14:40:12
tags:
---

## Spring

### Spring设计理念

当您了解框架时，不仅要了解框架的工作而且要遵循的原则很重要。以下是Spring框架的指导原则：

- 提供各个级别的选择。Spring使您可以尽可能延迟设计决策。例如，您可以在不更改代码的情况下通过配置切换持久性框架。对于许多其他基础架构问题以及与第三方API的集成也是如此。
- 适应不同的观点。Spring拥有灵活性，并且对如何完成事情没有意见。它从不同的角度支持广泛的应用程序需求。
- 保持强大的向后兼容性。对Spring的发展进行了精心管理，以使各个版本之间几乎没有重大更改。Spring支持精心选择的JDK版本和第三方库，以方便维护依赖于Spring的应用程序和库。
- 关心API设计。Spring团队投入了大量的思想和时间来制作直观，并在许多版本和很多年中都适用的API。
- 为代码质量设置高标准。Spring框架非常强调有意义、最新和准确的javadoc。它是极少数可以声明干净代码结构且程序包之间没有循环依赖关系的项目之一。

## spring优点

+ 开源免费框架（容器）
+ 轻量级非侵入式框架
+ 控制反转（IOC），面向切面编程（AOP）
+ 支持事务处理
+ 对框架整合的支持

***Spring是一个轻量级的控制反转IOC和面向切面编程AOP框架***

### Spring组成

Spring七大模块

+ Spring Core
+ Spring AOP
+ Spring ORM
+ Spring DAO
+ Spring Web
+ Spring Context
+ Spring Web MVC

### 了解

+ Spring Farmwork
+ Spring Boot
  + 一个快速开发的脚手架
  + 基于SpringBoot可以快速开发单个微服务
  + 约定大于配置（类似maven的思想）
+ Spring Cloud
  + SpringCloud是基于SpringBoot实现的

现在大多数公司都在使用SpringBoot进行快速开发，学习SpringBoot的前提是需要完全撞我Spring及SpringMVC

弊端：发展太久，违背了原有的理念！配置十分繁琐。

## 控制反转`IOC`

+ 之前程序是主动创建对象，控制权在程序手中。
+ 使用set注入后，程序不在具有主动性，而是变成了被动接受的对象。

控制反转IOC`Inversion of Control`是一种设计思想，DI依赖注入是实现IoC的一种方法，也有人认为DI是IoC的另一种说法。没有IoC的程序中，我们使用面向对象编程，对象的创建于对象间的依赖关系完全硬编码在程序中，对象的创建由程序自己控制，控制反转后将对象的创建转移给第三方，个人认为所谓控制反转：获得依赖对象的方式反转了。

**控制反转是一种通过描述（XML或注解）并通过第三方去生产或获取特定对象的方式。在Spring中实现控制反转的是IoC容器，其实现方法是依赖注入（Dependency Injection，DI）**

对象由Spring来创建、管理、装配。

### IOC创建对象的方式

+ 默认使用无参空构造创建对象
+ 如果需要使用有参构造，需要手动指定

### Spring配置

#### 别名

```xml
<alias name="hello" alias="helloAlias"/>

@Test
    public void sayTest(){
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
        Hello hello = (Hello) context.getBean("helloAlias");
        System.out.println(hello);
    }
```

#### import

可以将多个配置文件导入合并为一个

```xml
<import resource="pojo.xml"/>
```

### 依赖注入DI

+ 构造器注入
+ set注入
+ 其他方式注入

## Spring bean 作用域

| Scope                                                        | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [singleton](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-singleton) | (Default) Scopes a single bean definition to a single object instance for each Spring IoC container. |
| [prototype](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-prototype) | Scopes a single bean definition to any number of object instances. |
| [request](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-request) | Scopes a single bean definition to the lifecycle of a single HTTP request. That is, each HTTP request has its own instance of a bean created off the back of a single bean definition. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [session](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-session) | Scopes a single bean definition to the lifecycle of an HTTP `Session`. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [application](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-application) | Scopes a single bean definition to the lifecycle of a `ServletContext`. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [websocket](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-stomp-websocket-scope) | Scopes a single bean definition to the lifecycle of a `WebSocket`. Only valid in the context of a web-aware Spring `ApplicationContext`. |

## AOP

### 代理模式

+ 静态代理

+ 动态代理

  + 基于接口JDK动态代理
  + 基于类的实现cglib
  + Javassist基于字节码

JDK动态代理，

`java.lang.reflect.InvocationHandler`

`java.lang.reflect.Proxy`

   ```
package website.wanglu.proxy;

/**
 * @author luu
 */
public interface Rent {

    void rent();

}
   ```

```
package website.wanglu.proxy;

/**
 * @author: luu
 * @date: 2021-01-18 22:13
 **/
public class Landlord implements Rent {

    public void rent() {
        System.out.println("房东租房");
    }

}
```

```
package website.wanglu.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * @author: luu
 * @date: 2021-01-18 22:12
 **/
public class DynamicProxy implements InvocationHandler {

    private Rent rent;

    public DynamicProxy(Rent rent) {
        this.rent = rent;
    }

    public Object getLandlordProxy() {
        return Proxy.newProxyInstance(rent.getClass().getClassLoader(), rent.getClass().getInterfaces(), this);
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("before");
        return method.invoke(rent, args);
    }

}
```

```
package website.wanglu.proxy;

import org.junit.Test;

/**
 * @author: luu
 * @date: 2021-01-18 22:21
 **/
public class DynamicProxyTest {

    @Test
    public void rentTest() {
        DynamicProxy dynamicProxy = new DynamicProxy(new Landlord());
        Rent landlordProxy = (Rent) dynamicProxy.getLandlordProxy();
        landlordProxy.rent();
    }

}
```

### 什么是AOP

AOP`Aspect Oriented Programming`，面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是OOP的延续，是软件开发的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生泛型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑部分之间的耦合度降低，提高程序的可用性，同事提高了开发的效率。

### AOP在Spirng中的作用

提供声明式事务：运行用户自定义切面

+ 横切关注点：跨越应用程序多个模块的方法或功能。即是，与我们业务逻辑无关的，但是我们需要关注的部分，就是横切关注点。如：日志、安全、缓存、事务等...
+ 切面`Aspect`：横切关注点被模块化的特殊对象。是一个类。
+ 通知`Advice`：切面必须要完成的工作。是类中的一个方法。
+ 目标`Target`：被通知的对象
+ 代理`Proxy`：向目标对象应用通知之后创建的对象
+ 切入点`PoinCut`：切面通知执行的地点的定义
+ 连接的`JoinPoint`：与切入点匹配的执行点

```xml
xml配置：

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="userService" class="website.wang.service.UserServiceImpl"/>
    <bean id="userDao" class="website.wang.mapper.UserMapperImpl"/>
    <bean id="beforeAdvice" class="website.wang.log.BeforeLog"/>
    <bean id="afterAdvice" class="website.wang.log.AfterLog"/>

    <context:annotation-config/>

    <aop:config>
        <!--    切点    -->
        <aop:pointcut id="logPointcut" expression="execution(* website.wang.service.UserService.*(..))"/>
        <!--    织入    -->
        <aop:advisor advice-ref="beforeAdvice" pointcut-ref="logPointcut"/>
        <aop:advisor advice-ref="afterAdvice" pointcut-ref="logPointcut"/>
    </aop:config>

</beans>
```

```xml
使用xml定义切面：
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean id="userService" class="website.wang.service.UserServiceImpl"/>
    <bean id="userDao" class="website.wang.mapper.UserMapperImpl"/>
    <bean id="log" class="website.wang.log.Log"/>

    <aop:config>
        <!-- 切面 -->
        <aop:aspect ref="log">
            <!-- 切点 -->
            <aop:pointcut id="logPointcut" expression="execution(* website.wang.service.*.*(..))"/>
            <!-- 织入 -->
            <aop:before method="before"  pointcut-ref="logPointcut"/>
            <aop:after method="afterReturning" pointcut-ref="logPointcut"/>
        </aop:aspect>
    </aop:config>

</beans>
```

```java
代码方式：
package website.wang.log;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

/**
 * @author: luu
 * @date: 2021-01-19 11:27
 **/
@Component
@Aspect
public class LogAll {

    @Pointcut("execution(* website.wang.service.*.*(..))")
    public void pointCut() {

    }

    @Before("pointCut()")
    public void before() {
        System.out.println("===== before ====");
    }

    @After("pointCut()")
    public void after() {
        System.out.println("==== after ====");
    }

    @Around("pointCut()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("===== round before =====");
        Object proceed = joinPoint.proceed();
        System.out.println("===== round after =====");
        return proceed;
    }

}
```

## Spring整合myBatis

### MyBatis-Spring

MyBatis-Spring 会帮助你将 MyBatis 代码无缝地整合到 Spring 中。它将允许 MyBatis 参与到 Spring 的事务管理之中，创建映射器 mapper 和 `SqlSession` 并注入到 bean 中，以及将 Mybatis 的异常转换为 Spring 的 `DataAccessException`。 最终，可以做到应用代码不依赖于 MyBatis，Spring 或 MyBatis-Spring。

### Spring整合MyBatis方式一

1. 编写数据源配置
2. sqlSessionFactory
3. sqlSessionTemplate
4. 将myBatis的实现类注入到Spring中
5. 测试

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useUnicode=true&amp;characterEncoding=utf8&amp;useSSL=false"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
    </bean>

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="mybatis-config.xml"/>
    </bean>

    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory" />
    </bean>

    <bean id="personMapperImpl" class="website.wang.mapper.PersonMapperImpl">
        <property name="sqlSessionTemplate" ref="sqlSession"/>
    </bean>

</beans>
```



### Spring整合MyBatis方式二

```
package website.wang.mapper;

import org.mybatis.spring.support.SqlSessionDaoSupport;
import website.wang.bean.Person;

import java.util.List;

/**
 * @author: luu
 * @date: 2021-01-19 16:03
 **/
public class PersonMapperImpl2 extends SqlSessionDaoSupport implements PersonMapper {

    public int addPerson(List<Person> persons) {
        return getSqlSession().getMapper(PersonMapper.class).addPerson(persons);
    }

    public Person queryPersonById(long id) {
        return getSqlSession().getMapper(PersonMapper.class).queryPersonById(id);
    }

    public List<Person> queryPersonByName(String name) {
        return getSqlSession().getMapper(PersonMapper.class).queryPersonByName(name);
    }

    public int deletePerson(long id) {
        return getSqlSession().getMapper(PersonMapper.class).deletePerson(id);
    }

}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useUnicode=true&amp;characterEncoding=utf8&amp;useSSL=false"/>
        <property name="username" value="root"/>
        <property name="password" value="w926458L"/>
    </bean>

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="mybatis-config.xml"/>
    </bean>

    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory" />
    </bean>

    <bean id="personMapperImpl" class="website.wang.mapper.PersonMapperImpl2">
        <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
    </bean>

</beans>

```

### Spring声明式事务

+ 编程式事务
+ 声明式事务：AOP

### 使用注入映射器

```
<mybatis:scan base-package="org.mybatis.spring.sample.mapper" />
```

