---
layout: post
author: ᴢʜᴀɴɢ
title: "SpringBoot/Redis笔记"
date: 2023-05-21
music-id: 
permalink: /archives/2023-05-21/1
description: "SpringBoot/redis"
---

# SpringBoot笔记
## SpringBoot接口开发的常用注解有哪些？
```java
@Controller 标记此类是一个控制器,可以返回视图解析器指定的html页面,通过@ResponseBody可以将结果返回ison、 xml数据。
@RestController 相当于@ResponseBody加@Controller,实现rest接口开发,返回ison数据,不能返回html页面。
@RequestMapping 定义接口地址,可以标记在类上也可以标记在方法上,支持http的post、put、 get等方法。
@PostMapping 定义post接口，只能标记在方法上，用于添加记录，复杂条件的查询接口。
@GetMapping 定义get接口，只能标记在方法上，用于查询接口的定义。
@PutMapping 定义put接口，只能标记在方法上，用于修改接口的定义。
@DeleteMapping 定义delete接口，只能标记在方法上，用于删除接口的定义。
@RequestBody 定义在方法上，用于将json串转成java对象。
@PathVarible 接收请求路径中占位符的值.
@ApiOperation swagger注解，对接口方法进行说明。
@Api wagger注解，对接口类进行说明。
@Autowired 基于类型注入。
@Resource 基于名称注入,如果基于名称注入失败转为基于类型注入。
```
## redis
```java
穿透无中生有key，布隆过滤null隔离。
缓存击穿过期key，锁与非期解难题。
雪崩大量过期key，过期时间要随机。
面试必考三兄弟，可用限流来保底。
```
#### redis做为缓存，mysql的数据如何与redis进行同步呢？ （双写一致性）
```java
1. 介绍自己简历上的业务，我们当时是把文章的热点数据存入到了缓存中，虽然是热点数据,但是实时要求性并没有那么高，所以，我们当时采用的是异步的方案同步的数据
2. 我们当时是把抢券的库存存入到了缓存中，这个需要实时的进行数据同步，为了保证数据的强一致,我们当时采用的是redisson提供的读写锁来保证数据的同步
```
#### 介绍一下异步的方案(介绍一下redisson读写锁的这种方案)
```java
允许延时一致的业务，采用异步通知
① 使用MQ中间中间件，更新数据之后，通知缓存删除
② 利用canal中间件，不需要修改业务代码，伪装为mysql的一个从节点，canal通过读取binlog数据更新缓存
        
强一致性的，采用Redisson提供的读写锁
① 共享锁：读锁readLock，加锁之后，其他线程可以共享读操作
② 排他锁：独占锁writeLock也叫，加锁之后，阻塞其他线程读写操作
```