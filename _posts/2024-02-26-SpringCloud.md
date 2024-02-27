---
layout: post
author: ᴢʜᴀɴɢ
title: "SpringCloud笔记"
date: 2024-02-26
music-id: 
permalink: /archives/2024-02-26/1
description: "SpringCloud"
---

# SpringCloud笔记
### Eureka注册中心
```java
localhost:8080/user/变成了userservice/user/
给RestTemplate注解@LoadBalanced(userservice/user/请求到 Ribbon负载均衡 找eureka拉取服务)
负载均衡的是orderservice选一个userservice服务里的2个实例
```
### Nacos注册中心
#### 分级存储
```java
一级是服务，例如userservice
二级是集群，例如杭州或上海
三级是实例，例如杭州机房的某台部署了userservice的服务器
设置实例的集群属性修改application.yml文件，添加spring.cloud.nacos.discovery.cluster-name属性即可

要做配置的微服务名xxx:ribbon:NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRue 会优先选择本地集群

默认都是临时实例采用心跳监测 非临时实例nacos主动询问 ephemeral: false # 是否是临时实例

YAML配置热更新pattern:dateformat:yyyy/MM/dd
方式一:在@Value注入的变量所在类上添加注解@RefreshScope
方式二@Data@Component@ConfigurationProperties(prefix = "pattern")
public class PatternProperties {private String dateformat;}

多种配置的优先级：[服务名]-[环境].yaml >[服务名].yaml >本地配置
```
### Feign是一个声明式的http客户端(自带ribbon负载均衡) 代替RestTemplate
```java
在order-service的启动类添加注解开启Feign的功能@EnableFeignClients
@FeignClient ("userservice")
public interface UserClient {
    @GetMapping("/user/fid}")
    User findById(@PathVariable("id") Long id);
}

当定义的FeignClient不在SpringBootApplication的扫描包范围时，这些FeignClient无法使用。有两种方式解决：
方式一:指定FeignClient所在包@EnableFeignClients(basePackages = "cn.itcast.feign.clients")
方式二:推荐:指定FeignClient字节码@EnableFeignClients(clients = {UserClient.class})
```
### Gateway负载均衡相当于在很多orderservice里面选一个
#### Nacos的负载均衡是给orderservice选一个userservice
#### 网关的负载均衡是客户端请求到服务时发生的，而Feign的负载均衡是一个服务调另一个服务时发生的
```java
当过滤器的order值一样时，会按照 defaultFilter > 路由过滤器 > GlobalFilter的顺序执行。
```
### Docker
### 数据卷的作用：将容器与数据分离，解耦合，方便操作容器内数据，保证数据安全
```docker
数据卷操作：
docker volume create
docker volume ls
docker volume inspect查看详情
docker volume rm
docker volume prune删除未使用的
docker run ... -v html:/root/htm：把html数据卷挂载到容器内的/root/html这个目录中

目录挂载与数据卷挂载的语法类似：
-v [宿主机目录]:[容器内目录] 例-v /tmp/mysql/data:/var/lib/mysql
-v [宿主机文件]:[容器内文件] 
```

