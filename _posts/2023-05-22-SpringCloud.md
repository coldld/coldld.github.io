---
layout: post
author: ᴢʜᴀɴɢ
title: "SpringCloud"
date: 2023-05-22
music-id: 
permalink: /archives/2023-05-22/1
description: "SpringCloud"
---

# SpringCloud笔记
## Eureka注册中心
```java
localhost:8080/user/变成了userservice/user/
给RestTemplate注解@LoadBalanced(userservice/user/请求到 Ribbon负载均衡 找eureka拉取服务)
负载均衡的是orderservice选一个userservice服务里的2个实例
```
## Nacos注册中心
### 分级存储
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
## Feign是一个声明式的http客户端(自带ribbon负载均衡) 代替RestTemplate
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
## Gateway负载均衡相当于在很多orderservice里面选一个
#### Nacos的负载均衡是给orderservice选一个userservice
#### 网关的负载均衡是客户端请求到服务时发生的，而Feign的负载均衡是一个服务调另一个服务时发生的
```java
当过滤器的order值一样时，会按照 defaultFilter > 路由过滤器 > GlobalFilter的顺序执行。
```
## Docker
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

### SpringAMQP
提供了模板来发送和接收消息。包含两部分，其中spring-amqp是基础抽象，spring-rabbit是底层的默认实现。
利用RabbitTemplate的convertAndSend方法
```java
消费者 接收消息
@RabbitListener(queues="队列名")

修改application.yml文件,设置preFetch这个值,可以控制预取消息的上限:
preFetch: 1 # 每次只能获取一条消息，处理完成才能获取下一个消息

交换机:FanoutExchange的使用 广播发送
在consumer服务常见一个类，添加@Configuration注解，并声明FanoutExchange、Queue和绑定关系对象Binding
在consumer服务的SpringRabbitListener类中，添加两个方法，分别监听fanout.queue1和fanout.queue2
在publisher服务发送消息到FanoutExchange 以前发送到队列 现在发送到交换机
交换机:Direct Exchange会将接收到的消息根据规则路由到指定的Queue,因此称为路由模式(routes)
可以模拟广播 但是要指定key
@RabbitListener(bindings = @QueueBinding(
value = @Queue(name = "direct.queue1"),
exchange = @Exchange(name = "itcast.direct", type = ExchangeTypes.DIRECT),
key = {"red", "blue"}))
public void listenDirectQueue1(String msg){
System,out.println("消费者1接收到Direct消息：msg
交换机:TopicExchange与DirectExchange类似，区别在于routingKey必须是多个单词的列表，并且以.分割。
Queue与Exchange指定BindingKey时可以使用通配符：
#代指0个或多个单词   *代指一个单词

消息转换器
SpringAMQP中消息的序列化和反序列化是怎么实现的？
利用MessageConverter实现的，默认是JDK的序列化(建议用自定义的MessageConverter)
例
在consumer服务定义MessageConverter:
return new Jackson2JsonMessageConverter();
```
## ES
```java
# 创建文档
POST/索引库名/_doc/文档id{json文档}
# 查询文档
GET /索引库名/_doc/文档id
# 删除文档
DELETE /索引库名/_doc/文档id
# 修改文档
全量修改，会删除旧文档，添加新文档
PUT /索引库名/_doc/文档id
{
"字段1": "值1"，
"字段2"："值2"，
//...略
}
增量修改，修改指定字段值(局部修改)
POST /索引库名/_update/文档id

RestClient
用来操作ES。这些客户端的本质就是组装DSL语句,通过http请求发送给ES。
```

