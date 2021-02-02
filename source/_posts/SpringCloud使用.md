---
title: SpringCloud使用
date: 2021-01-25 14:57:07
tags:
---

## 微服务架构的核心问题

+ 服务很多，客户端如何访问，服务网关
+ 服务很多，服务间如何通信
+ 服务很多，夫如何发现和治理
+ 服务挂掉怎么办

## SpringCloud

SpringCloud生态，使用SpringBoot技术提供微服务架构的解决方案。

+ SpringCloud NetFlix 一站式解决方案
  + api网关 zuul组件
  + Feign --HttpClient --Http通信方式，同步阻塞
  + 服务注册发现Eureka
  + 熔断机制：Hystrix
+ Apache Dubbo Zookeeper 半自动解决方案
  + api：没有，需要第三方组件或自己实现
  + Dubbo
  + Zookeeper
  + 熔断机制：没有，需要第三方组件
+ SpringCloud Alibaba 一站式解决方案
+ 新概念：服务网格 -- service Mesh 
  + istio

> 核心知识点
>
> 1. API
> 2. HTTP、RPC
> 3. 注册和发现
> 4. 熔断机制

## 微服务技术栈

| 微服务条目          | 落地技术                                                     |
| ------------------- | ------------------------------------------------------------ |
| 微服务开发          | SpringBoot、Spring、SpringMVC                                |
| 服务配置与管理      | Netflix的Archaius、阿里的Diamond等                           |
| 服务注册与发现      | Eureka、Consul、Zookeeper等                                  |
| 服务调用            | Rest、RPC、gRPC                                              |
| 服务熔断器          | Hystrix、Envoy等                                             |
| 负载均衡            | Nginx、Ribbon等                                              |
| 服务接口调用        | Feign等                                                      |
| 消息队列            | Kafka、RabbitMQ、ActiveMQ等                                  |
| 服务配置中心管理    | SpringCloudConfig、Chef等                                    |
| 服务路由（API网关） | Zuul等                                                       |
| 服务监控            | Zabbix、Nagios、Metrics、Specatator等                        |
| 全链路追踪          | ZipKin、Brave、Dapper等                                      |
| 服务部署            | Docker、OpenStack、Kubernets等                               |
| 数据流操作开发包    | SpringCloudStream（封装与Redis、Rabbit、Kafka等发送接收消息） |
| 事件消息总线        | SpringCloud Bus                                              |

## SpringCloud是什么

## Eureka服务注册中心

### CAP原则

+ `Consistency` 强一致性
+ `Availability`可用性
+ `Partition tolerance`分区容错性（集群必须）

只能取其二

## Ribbon

客户端的负载均衡工具

LB`LoadBalance`负载均衡

负载均衡算法

## Fegin

负载均衡

类似Dubbo方式，通过注解形式将请求映射到ServiceInterface，之后可以直接调用。

## Hystrix

+ 服务熔断：熔断机制是对应雪崩效应的一种微服务链路保护机制。当链路的某个微服务不可用或者响应时间过长时，会进行服务降级，进而熔断该节点微服务的调用，快速返回错误的响应信息。当检测到该节点微服务调用响应正常后恢复调用链路。SpringCloud中的熔断机制通过Hystrix实现。Hystrix会监控微服务间调用的状况，当失败的调用到一定阈值，缺省是5s内20次失败，就会触发熔断机制。熔断机制的注解是@HystrixCommand

+ 服务降级：

对等应用阿里出的Sentinel

## dashboard流量监控

## Zuul路由网关

+ Zuul包含了对请求的路由和过滤两个主要的功能
+ 路由功能负责外部请求转发到具体的微服务实例上，是实现外部统一入口的基础
+ 过滤功能主要对请求处理过程进行干预，实现请求校验，服务聚合等功能。

## SpringCloud Config

微服务意味着要将单体应用中的业务拆分为一个个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务，由于每个服务都需要必要的配置信息才能运行，所以一套集中式的，动态的配置管理设施必不可少。SpringCloud提供了ConfigServer来解决这个问题，我们每一个微服务自己带一个application.yml，那上百的配置文件要修改起来，很庞大。