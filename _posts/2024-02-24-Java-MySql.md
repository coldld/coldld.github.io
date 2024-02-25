---
layout: post
author: ᴢʜᴀɴɢ
title: "Java MySql 笔记"
date: 2024-02-24
music-id: 
permalink: /archives/2024-02-24/1
description: "Java MySql"
---

# Java笔记
![Collection](https://aroucc.oss-cn-hangzhou.aliyuncs.com/images/Collection.png)
![IO](https://aroucc.oss-cn-hangzhou.aliyuncs.com/images/IO%E6%B5%81%E4%BD%93%E7%B3%BB.png)
![lambda](https://aroucc.oss-cn-hangzhou.aliyuncs.com/images/%E5%8C%BF%E5%90%8D%E5%86%85%E9%83%A8%E7%B1%BB%E6%88%90lambda.png)
![mybatis_2._and_1.](https://aroucc.oss-cn-hangzhou.aliyuncs.com/images/mybatis.png)
```java
null表示对象为空，isEmpty表示值为空

- ArrayList:数组:查询快 增删慢
- LinkedList:双链表:查询慢 增删相对快 首尾元素CURD速度极快 [做队列(排队买票),栈(弹夹)适合]
- HashSet:哈希表:数组(默认长度16加载因子0.75)+链表+(JDK8后链表长度>8,数组长度>=64自动链表转成红黑树)

集合并发修改异常:迭代器遍历使用自己的删除方法.for循环:倒着遍历删除,i--,不能使用增强for循环
```

```java
Java基本类型占用的字节数：
1字节： byte , boolean
2字节： short , char
4字节： int , float
8字节： long , double
编码与中文：
Unicode/GBK： 中文2字节
UTF-8： 中文通常3字节，在拓展B区之后的是4字节
```

```java
StringBuilder线程不安全,StringBuffer线程安全。
对于字符串相关的操作，如频繁的拼接、修改等，建议用StringBuilder，效率更高!
操作字符串较少，或不需要操作，以及定义字符串变量，还是建议用String。
```
### Mybatis参数占位符
```java
mapper里sql语句中的参数占位符
#{...} 执行SQL时，会将#{...}替换为?，生成预编译SQL，会自动设置参数值。 参数传递时使用。
${...}  拼接SQL，直接将参数拼接在SQL语句中，存在SQL注入问题。对表名、列表进行动态设置时使用。
        
模糊查询 where name like '%${name}%' 不能使用#{name} 因为字符串中不能有问号
字符串拼接函数 concat('%',#{name},'%')
```
### Lombok
```java
Lombok 能通过注解的形式自动生成构造器 需要引入依赖
@EqualsAndHashCode 根据类所拥有的非静态字段自动重写 equals方法和 hashCode方法
@Data 提供了更综合的生成代码功能(@Getter + @Setter + @ToString + @EqualsAndHashCode)
@NoArgsConstructor 为实体类生成 无参构造
@AllArgsConstructor 为实体类生成除了static修饰的字段之外 全参构造
```
### Mybatis主键返回@Options
```java
@Options(keyProperty = "id", useGeneratedKeys = true) 会自动将生成的主键值，赋值给emp对象的id属性
@Insert("insert into emp(... , ...)values(... , ...)
void insert(Emp emp);
```
### Mybatis开启的驼峰camel命名自动映射开关
```java
mybatis.configuration.map-underscore-to-camel-case = true
```
### Mybatis 动态SQL
```xml
<where>会自动去除条件里开头的 and 和 or
    <if test="name != null"> 
        and ...
    </if>
</where>

<set>会删掉额外的逗号（用在update语句中)</set>

<foreach collection="ids遍历的集合名称" item="id遍历出来的元素" 
         separator=",分隔符" open="(遍历开始前拼接的SQL片段" close=")遍历结束后拼接的SQL片段">
    #{id}
</foreach>

<sql id="XXX引用"></sql>先定义sql语句 再调用<include refid="XXX引用"/>
```
# MySql笔记
```sql
[database=schema]

show databases/tables;
desc 表名; -- 查询表结构
show create table 表名; -- 查询建表语句

select database(); -- 查询当前数据库
create database [if not exists] mydb;
drop database [if exists] mydb;
use mydb;
```
### DDL表结构操作
```sql
create table 表名(id int primary key autu_increment comment 'ID唯一标识', ... , ...)[comment '表注释'];
not null -- 非空约束
unique -- 唯一约束 唯一不重复
primary key -- 主键约束 (autu_increment自增) 主键是一行数据的唯一标识，非空且唯一 
default -- 默认约束  保存数据时，如果未指定该字段值，则采用默认值
foreign key-- 外键约束 物理外键(影响增删改效率 建议逻辑外键)让两张表的数据建立连接，保证数据的一致性和完整性 
-- (仅用于单节点数据库，不适用与分布式、集群场景。容易引发数据库的死锁问题，消耗性能)
-- constraint 外键名称 foreign key (外键字段名) references 主表(字段名)
例子 让两张表的数据建立连接
alter table tb_emp add constraint tb_emp_fk_dept_id foreign key (dept_id) references tb_dept (id) ;


tinyint 字节1 [signed(有符号)范围(-128,127)] [unsigned(无符号) 范围(0,255)]
-- smallint 字节2
-- mediumint 字节3
int 字节4
bigint 字节8
float 字节4 float(5,2)5是整个数字长度，2是小数个数
double 字节8
decimal 和java里BigDecimal一样 decimal(8,2)8是整个数字长度，2是小数个数
char 字节范围(0,255) 定长字符串 char(10):[性能高浪费空间]最多只能存10个字符,不足10个字符,占用10个字符空间
varchar 字节范围(0,65535) 变长字符串 varchar(10):[性能低节省空间]最多只能存10个字符,不足10个字符,按照实际长度存储

添加字段: alter table 表名 add 字段名 类型(长度) [comment 注释] [约束];
修改字段类型: alter table 表名 modify 字段名 新数据类型(长度);
修改字段名和字段类型: alter table 表名 change 旧字段名 新字段名 类型(长度) [comment 注释] [约束];
删除字段: alter table 表名 drop column 字段名;
修改表名: rename table 表名 to 新表名;
删除表名: drop table [if exists] 表名;
```
### DML增删改
```sql
insert into tb_emp (username,name,create_time) values ('aaa","插入",now()), ('piliang","批量",now());
update tb_emp set name = '更新' , update_time = now() where id = 1; -- 不加where全部更新
delete from tb_emp where id = 1; -- 不加where全部删除此表数据
```
### DQL查询
```sql
select (distinct去重记录) 字段列表 (as 别名 可省略) ( * 性能低?) ( cout(*)性能好)
from 表名 -- 表名,表名 多表查询用,隔开   笛卡尔积:笛卡尔乘积是指在数学中,两个集合(A集合和B集合)的所有组合情况。
where 条件
    between and 在某个范围之内(含最小、最大值)
    in(..) 在in之后列表中的值(多选一 单个符合就行)
    like 模糊匹配(_匹配单个字符，%匹配任意个字符)
group by 分组字段 例1 例3 分组查询select返回两类字段，一类是分组字段例如gender，另一类是聚合函数例如 count(*)
having 分组后条件
order by 排序字段 例2多字段排序用 ,  默认升序 -- ASC：升序   DESC：降序
limit 分页(起始索引, 查询记录数) -- 第一页=起始索引0 每页展示5条记录:  0,5  
-- 起始索引 = (页码 - 1）* 每页展示记录数
-- 如果查询的是第一页数据，起始索引可以省略，直接简写为 limit 5

例1
根据性别分组，统计男性和女性员工的数量
    select gender,count(*) from tb_emp group by gender;
先查询入职时间在 '2015-01-01'（包含）以前的员工，并对结果根据职位分组 ， 获取员工数量大于等于2的职位
    select job,count(*) from tb_emp where entrydate <= '2015-01-01' group by job having count(*) >=2;

where与having区别
1. 执行时机不同: where 是分组之前进行过滤，不满足where条件，不参与分组;而 having 是分组之后对结果进行过滤。
2. 判断条件不同: where 不能对聚合函数进行判断，而 having 可以。
执行顺序: where > 聚合函数 > having

例2
根据 入职时间 升序，入职时间相同，再按照 更新时间 降序
select * from 表 order by 入职时间 , 更新时间 desc

例3
员工性别信息的统计 count(*)
if(条件表达式，true取值，false取值)
select if(gender = 1，'男'，'女') 性别，count(*)from tb_emp group by gender

员工职位信息的统计
case 表达式 when 值1 then 结果1 when 值2 then 结果2 ... else ... end
select
    (case job when 1 then "啊" when 2 then "我" when 3 then "额" else "无" end ),
    count(*)
from tb_emp group by job;
```
### 连接查询
```sql
### 内连接：相当于查询A、B交集部分数据
查询员工的姓名，及所属的部门名称（隐式内连接实现）
select tb_emp.name,tb_dept.name from tb_emp,tb_dept where tb_emp.dept_id = tb_dept.id;
起别名 select e.name, d.name from tb_emp e, tb_dept d where e.dept_id = d.id;
查询员工的姓名，及所属的部门名称（显式内连接实现)
select tb_emp.name,tb_dept.name from tb_emp [inner 可省略] join tb_dept on tb_emp.dept_id = tb_dept.id;

### 外连接：
#### 左外连接：查询左表所有数据(包括两张表交集部分数据) select 字段列表 from 表1 left [outer] join 表2 on 连接条件;
查询员工表 所有 员工的姓名，和对应的部门名称（左外连接)
select e.name, d.name from {tb_emp e left join} tb_dept d on e.dept_id = d.id

#### 右外连接：查询右表所有数据(包括两张表交集部分数据)
查询部门表 所有 部门的名称，和对应的员工名称（右外连接)
select e.name,d.name from tb_emp e {right join tb_dept d} on e.dept_id = d.id
```
### 子查询
#### SQL语句中嵌套select语句,称为嵌套查询,又称子查询。
```sql
### 标量子查询(只有一个值)
select * from tb_emp where dept_id = (select id from tb_dept where name = '一部');

### 列子查询 返回的结果是一列（可以是多行) 常用的操作符： in 、 not 、 in 等
一行XXX1 -- 相当于数据库里查出来一列两行
两行XXX2 -- 相当于数据库里查出来一列两行
select id from tb_dept where name='一部' or name='二部'; -- 输出结果(2 , 3)
select * from tb_emp where dept_id in (select id from tb_dept where name = '一部' or name = '二部');

### 行子查询 返回的结果是一行（可以是多列 例如: 一列XXX1 两列XXX2 ）。常用的操作符： = 、 ◇ 、 in 、 not in
select entrydate,job from tb_emp where name = '姓名';-- 输出结果('2000-01-01',2) 放入下面sql语句()中即可
select * from tb_emp where (entrydate,job)=('2000-01-01',2);

### 表子查询 返回的结果是多行多列，常作为临时表常用的操作符： in
查询入职日期是"2006-01-01"之后的员工信息
select * from tb_emp where entrydate > '2006-01-01';
查询这部分员工信息及其部门名称 - tb_dept
select e.* , d.name from (select * from tb_emp where entrydate > '2006-01-01') e , tb_dept d where e.dept_id = d.id;
```
### 索引(index)
#### 帮助数据库 高效获取数据 的 数据结构。相当于一本书的目录
#### mysql默认索引结构B+Tree(多路平衡搜索树)矮胖矮胖的树。叶子节点是双向链表便于排序
```sql
primary key 主键字段，在建表时，会自动创建主键索引。性能最高
unique 添加唯一约束时，数据库实际上会添加唯一索引。
创建 create [unique] index 索引名 on 表名(字段名,...);
例 create index idx_sku_sn on tb_sku(sn);
查看 show index from 表名;
删除 drop index 索引名 on 表名;
索引会占用存储空间。索引提高了查询效率，同时也降低了insert,update,delete的效率。
```

```
取本地IP接口
https://api.live.bilibili.com/xlive/web-room/v1/index/getIpInfo
https://ipv4.gdt.qq.com/get_client_ip
```