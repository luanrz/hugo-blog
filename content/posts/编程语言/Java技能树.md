---
title: Java技能树
date: 2022-06-20 15:34:51
categories:
- 编程语言
- Java
tags:
- Java
---

梳理Java开发所需的技能点，用于指导后续的学习路线，并提供系统性的查漏补缺方式。

<!--more-->

```plantuml

@startmindmap Java技能树

* Java技能

** Java基础
*** IO
*** 集合
*** 多线程
**** 理论基础
***** Java内存模型(JMM)
**** 锁
***** 队列同步器(AQS)
***** Lock接口及其实现类与支持类
**** 并发工具
***** 集合：ConcurrentHashMap、ConcurrentLinkedQueue、BlockingQueue相关类
***** 工作窃取：ForkJoin相关类
***** CAS原子操作：Atomic相关类
***** 流程控制与数据交换：CountDownLatch、Semaphore、Exchanger
**** 线程池
*** JVM
**** JVM内存结构
**** 垃圾回收(GC)
**** 类加载机制
**** 性能调优

** 理论基础
*** 设计模式
*** 数据结构与算法

** 开发框架
*** Spring(SpringMVC、SpringBoot、SpringCloud、SpringDataJPA)
*** 数据库(Hibernate、Mybatis)
*** 服务调用(Dubbo)

** 数据库
*** 数据库基础
**** 事务
*** MySql
*** Oracle

** 中间件
*** 缓存
**** Redis
**** Memcache
**** Guava
**** Ehcache
*** 消息
**** Kafka
**** RabbitMQ
*** 部署
**** Apache
**** Nginx
**** Tomcat
**** Jboss
*** 搜索引擎
**** Elasticsearch
*** 协调服务
**** ZooKeeper

** 运维
*** Linux
*** 脚本
**** Python
**** Shell
*** 容器
**** Docker
**** Kubernetes
*** CI/CD
**** Jenkins

** 分布式架构
*** 基础概念
**** CAP
***** 一致性(Consistency)
***** 可用性(Availability)
***** 分区容错性(Partition tolerance)
**** 三高
***** 高并发
***** 高性能
***** 高可用
*** 分布式服务(微服务治理)
**** 服务注册与发现
***** Nacos
***** Eureka
**** 服务调用
***** Feign
***** RPC(Dubbo)
**** 负载均衡
***** SpringCloudLoadBalancer
***** Ribbon
**** 配置管理
***** SpringCloudConfig
***** Nacos
**** 网关
***** SpringCloudGateway
***** Zuul
**** 熔断降级
***** SpringCloudCircuitBreaker
***** Sentinel
***** Hystrix
***** Resilience4j
*** 分布式缓存
*** 分布式事务
*** 分布式锁

** 工具
*** Git
*** Maven

@endmindmap

```

上图中，相同的概念会尽量放在一起，但根据关注点的不同，相同的概念也可能会体现在不同的节点下面（如：Redis可以同时出现在中间件与分布式缓存中，中间件中的Redis更关注Redis的基础特性，而分布式缓存更关注Redis的分布式特性）。

有部分概念比较相近，需要注意它们之间的区别，如：多线程下的`Java内存模型`与JVM下的`JVM内存结构`，两者不是同一个概念。

同时，由于技术的快速发展与个人视野的局限性，上图内容还不是很完善，后续将按需动态更新。

> 参考文档

1. 《深入理解Java虚拟机第3版 周志明 著》
2. 《Java并发编程的艺术 方腾飞 魏鹏 程晓明 著》