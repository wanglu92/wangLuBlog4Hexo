---
title: SpringBoot使用
date: 2021-01-21 16:08:39
tags:
---

## SpringBoot优点

+ 为所有Spring开发更快速搭建环境
+ 开箱即用，提供各种默认配置来简化项目配置
+ 内嵌式容器简化web项目
+ 没有荣誉代码生成和XML配置的要求

## 微服务阶段

## 微服务

将服务模块化，部署到不同的机器上，负责提供自己能力的接口。

## 自动配置原理

+ `spring-boot-dependencies`：核心依赖在父工程中，父工程中配置了各种无冲突的依赖版本。
+ 启动器:SpringBoot的启动场景，需要什么开发场景找到对应的启动器引入就可以。

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

+ 主程序

  + 注解`@SpringBootApplication`

  + 将SpringBoot项目启动

  + ```
    public static void main(String[] args) {
            SpringApplication.run(SpringbootexerciesApplication.class, args);
        }
    ```

  + `META-INF/spring.factories.`自动配置的核心文件

> + SpringBoot启动会加载大量的自动配置类
> + 我们看我们需要的功能有没有在SpringBoot默认写好的自动配置类当中
> + 我们再来看这个自动配置类中到底配置了哪些组件；只要我们要用的组件存在其中，我们就不需要再手动配置了
> + 给容器中自动配置类添加组件的时候，会从properties类中获取某些属性，我们只需要在配置文件中指定这些属性的值即可。

## run()运行原理



## yaml配置









