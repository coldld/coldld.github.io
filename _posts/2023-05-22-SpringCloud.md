---
layout: post
author: ᴢʜᴀɴɢ
title: "SpringCloud笔记"
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
### 服务降级
是服务自我保护的一种方式,或者保护下游服务的一种方式,用于确保服务不会受请求突增影响变得不可用，确保服务不会崩溃
服务降级虽然会导致请求失败，但是不会导致阻塞。  
①：在heima-leadnews-feign-api编写降级逻辑  
②：远程接口中指向降级代码  
③：客户端开启降级  

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
### Kafka
追求高吞吐量,适合产生大量数据的互联网服务的数据收集业务
### RocketMQ
可靠性要求很高的金融互联网领域,稳定性高,经历了多次阿里双11考验
### RabbitMQ
性能较好,社区活跃度高,数据量没有那么大,优先选择功能比较完备的RabbitMQ
## kafka
```dockerfile
先安装zookeeper
docker run -d --name zookeeper -p 2181:2181 zookeeper:3.4.14

再安装kafka
docker run -d --name kafka \
--env KAFKA_ADVERTISED_HOST_NAME=192.168.200.130 \
--env KAFKA_ZOOKEEPER_CONNECT=192.168.200.130:2181 \
--env KAFKA_ADVERTISED_LISTENERS-PLAINTEXT://192.168.200.130:9092 \
--env KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 \
--env KAFKA_HEAP_OPTS="-Xmx256M -Xms256M" \
--net=host wurstmeister/kafka:2.12-2.3.1

云主机除外,必须映射 -p 9092:9092
---net=host,直接使用容器宿主机的网络命名空间,即没有独立的网络环境。它使用宿主机的ip和端口
```
### Kafka高可用设计
1. 集群
2. 备份机制(Replication)
   Kafka中消息的备份又叫做副本(Replica)  
   Kafka 定义了两类副本：  
   领导者副本(Leader Replica)  
   追随者副本（Follower Replica)  
   ISR (in-sync replica)需要同步复制保存的follower
   #### 如果leader失效后，需要选出新的leader，选举的原则如下：  
   第一:选举时优先从ISR中选定，因为这个列表中follower的数据是与leader同步的  
   第二:如果ISR列表中的follower都不行了,就只能从其他follower中选取  
   #### 极端情况，就是所有副本都失效了，这时有两种方案  
   第一：等待ISR中的一个活过来，选为Leader，数据可靠，但活过来的时间不确定一  
   第二：选择第一个活过来的Replication，不一定是ISR中的，选为leader，以最快速度恢复可用性，但数据不一定完整 
   
生产者配置
```java
//ack配置 消息确认机制 默认1
prop.put(ProducerConfig.ACKS_CONFIG, "all");
//重试次数10次
prop.put(ProducerConfig.RETRIES_CONFIG, 10);
//数据压缩
prop.put(ProducerConfig.COMPRESSION_TYPE_CONFIG,"snappy");
```
消费者配置  
偏移量提交方式 同步和异步组合提交
## RabbitMQ
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
## ES(elasticsearch)
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
```
### RestClient
用来操作ES。这些客户端的本质就是组装DSL语句,通过http请求发送给ES。

## Freemarker指令语法
`gt代替>, FreeMarker会把>解释成FTL标签的结束字符,可使用括号避免这种情况,如:<#if (x>y)>`
```markdown
遍历list
<#list stus as stu> 
    <tr>
        <td>${stu_index + 1}</td> 
        <td>${stu.name}</td> 
        <td>${stu.age}</td> 
        <td>${stu.money}</td>
    </tr>
</#list>

遍历map
<#list stuMap?keys as key>
    <tr>
        <td>${key_index + 1}</td> 
        <td>${stuMap[key].name}</td> 
        <td>${stuMap[key].age}</td> 
        <td>${stuMap[key].money}</td>
    </tr>
</#list>

if指令
<#if stu.name='小红'>...<#else >...
</#if>

空值处理
用法为:variable??,如果该变量存在,返回true,否则返回false
例:为防止stus为空报错可以加上判断如下:
<#if stus??>
<#list stus as stu></#list>
</#if>

缺失变量默认值使用"!"
使用！要以指定一个默认值，当变量为空时显示默认值
例: $fname!'')表示如果name为空显示空字符串。
如果是嵌套对象则建议使用（）括起来
例: $f(stu.name)!''}表示,如果stu或name为空默认显示空字符串。

内建函数语法格式:变量 + ? + 函数名称
1. 集合的大小${集合名?size}
2. 日期格式化
显示年月日： ${today?date}
显示时分秒: ${today?time}
显示日期+时间: ${today?datetime}
自定义格式化: ${today?string("yyyy年MM月")}

3. model.addAttribute("point", 102920122); 输出-> 102,920,122
不想每三位分隔的数字的话${point?c}

4. 将json字符串转成对象text?eval
例子：其中用到了assign标签，assign的作用是定义一个变量。
<#assign text="{'bank':'工商银行','account': '10101920201920212'}" />
<#assign data=text?eval />
开户行：${data.bank} 账号:${data.account}
```
## minIO
```dockerfile
docker run -p 9000:9000 --name minio -d --restart=always -e "MINIO_ACCESS_KEY=minio" -e "MINIO_SECRET_KEY=minio123" -v /home/data:/data -v /home/config:/root/.minio minio/minio server /data
```

