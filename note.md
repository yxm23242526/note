# Go

`string`类型不可以被修改

字符串太长时，需要将 `+`保留到上一行

只有`float32`和`float64`

显式转换时，变量本身的类型不会改变，改变的是变量存储的值

基本数据类型转字符串 `fmt.Sprintf`和`strconv`包

首字母小写是私有的，首字母大写是公有的

长度是数组的一部分，函数传递数组时，长度也需要保持一致

如果一个类型实现了String()方法，那么fmt.Println默认会调用这个变量的String()进行输出

定义接口后，只要实现接口的所有方法就代表实现接口了





# 服务器相关

公网IP：47.102.128.67

管理员模式 `su`

账号`root` 密码`Yxm980918.`

- `ps -ef`：查看所有进程
- `ps -aux`：查看所有进程
- `ps -ef | grep tomcat`：查看指定进程

`cd /usr/local/bin`

`redis-cli -h 47.102.128.67 -p 6379 -a 123456`

# Java

## 线程池相关

参考文章：

> [【多线程系列-01】深入理解进程、线程和CPU之间的关系_cpu和线程的关系-CSDN博客](https://blog.csdn.net/zhenghuishengq/article/details/131714191)

---

CPU和线程的关系:

​	每个进程都要使用共享的cpu，目前主流的CPU都是多核的，线程是CPU调度的最小单位，也就是说，一个核心CPU只能运行一个线程，**但Intel引入这个超线程技术之后，产生了逻辑处理器的概念，使得一个cpu中，可以执行两个线程，从而让cpu数和线程数达到 1:2 的关系** 

---

上下文切换：

​	指的是**从一个进程切或线程切换到另一个进程或者线程的切换**

---

IO密集型和CPU密集型的线程池的线程数量设置：

- IO密集型，一般认为是逻辑处理器（n）的2倍，即2n， 因为此时cpu使用较少，可以频繁切换上下文
- CPU密集型，n + 1，保证CPU不频繁切换

`ThreadPoolExecutor`有7个参数

> `corePoolSize` 核心线程池大小
>
> `maximumPoolSize` 线程池能创建线程的最大个数
>
> `keepAliveTime` 空闲线程存活时间
>
> `unit` 时间单位，为`keepAliveTime`指定时间单位
>
> `workQueue` 工作队列
>
> `threadFactory` 创建线程的工厂类
>
> `handler` 饱和策略



------------

```java
// Runnable代码块 
Thread thread = new Thread(new Runnable() {    
    @Override    public void run() {        
        System.out.println("Hello Man!");    
    } }); 
// 使用Lambda表达式简化 
Thread thread = new Thread(() -> System.out.println("Hello Man!"));
```

这里能替换为lamda表达式的原因是，java8引入的新特性，把那些仅有一个抽象方法的接口称为**函数式接口**。如果一个接口被@FunctionalInterface注解标注，就是函数式接口，Runnable就是其中之一

又允许lamda表达式直接作为这个接口的实现类

## JVM

参考文章

> [JVM的内存区域划分_jvm内存区域划分-CSDN博客](https://blog.csdn.net/m0_67995737/article/details/130064952)

# MySQL

2024.4.18

mysql的事务隔离级别：

| 传播性                                     | 描述                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| serializable                               | 同一时间只有一个事务在跑，性能差，安全性高                   |
| repeatable-read（可重复度，默认）          | 并发执行Sql时，其他sql的修改对当前不可见，只有当前自己提交了才能看见 |
|                                            | 幻读：Asql查询 X， 随即Bsql插入X，Asql依然查不到的情况下插入X，会报错 |
| read commited（不可重复读、幻读）          | 可以看到其他事务的修改，一个相同的sql也就可能看到多个结果    |
| read uncommited （不可重复读、幻读、脏读） | 可以读到其他事务没有提交的数据                               |

随着隔离级别的升高，性能会降低





聚簇索引和非聚簇索引

> ```mysql
> mysql> create table user(
>     -> id int(10) auto_increment,
>     -> name varchar(30),
>     -> sex tinyint(4),
>     -> type varchar(8),
>     -> primary key (id),
>     -> index idx_name (name)
>     -> )engine=innodb charset=utf8mb4;
> ```
>
> 主键id是聚簇索引，其叶子节点存储的是**对应行记录**的数据
>
> name是普通索引，即非聚簇索引，其叶子节点存储的是**聚簇索引**的的值
>
> 回表：先通过普通索引找到聚簇索引，再通过聚簇索引找到行
>
> 索引覆盖：一次查询可以查到所有需要的列

聚簇索引（InnoDB)  将数据存储和索引放到了一块，索引结构的叶子结点保留了行数据， 以及有辅助表，如按user_name排序的表，存放的是主键ID

非聚簇索引（MyISAM) 数据和索引分开，两张表

联合索引是因为多个索引是按A排序完再按B，再C等等，所以需要遵循最佳左前缀法则，否则直接查B是乱序，导致索引失效

```Java
//伪代码
public void compareTo(A, B, C){
 	if (A == this.A){
  		if (B == this.B){
  			return C < this.c
  		}
 		return B < this.B
 	}
 	return A < this.A
}
```



持久性如何实现：redo日志+刷盘

> 将修改的数据库页先记录到redo日志中，并保证在刷盘之前，这样如果意外崩溃，InnoDB首先读取redo日志还原

`binlog`:实现数据恢复、主从节点复制等

`binlog`和`redolog`的区别：

> - 产生层次：binlog是MySQL数据库的上层服务层产生的，而redolog是InnoDB存储引擎层产生的。这意味着binlog是MySQL服务器层面的日志，而redolog是InnoDB存储引擎特有的日志。
> - 记录内容：binlog是逻辑日志，记录的是对数据库的所有修改操作，如SQL语句的原始逻辑。而redolog是物理日志，记录的是每个数据页的修改，即“在某个数据页上做了什么修改”。
> - 写入方式：binlog是追加写入，即写完一个日志文件再写下一个日志文件，不会覆盖使用。redolog是循环写入，日志空间的大小是固定的，会覆盖使用。
> - 用途：binlog一般用于主从复制和数据恢复，不具备崩溃自动恢复的能力。redolog在服务器发生故障后重启MySQL时，用于恢复事务已提交但未写入数据表的数据。

------------

## 关键字

- `COUNT(1)`: 此查询返回的是结果集中的行数，不关心具体的列内容，因此使用常数1。
  在很多数据库系统中，这种方式被优化为与 `SELECT COUNT(*)` 相同的性能水平，因为数据库引擎通常忽略括号内的内容。
- `COUNT(*)`：统计整个表的行数，不考虑是否有NULL值。
  通常优于 `COUNT(id)`，因为它不需要关心具体的列，且现代数据库引擎会对其进行特殊优化。
- `COUNT(列)` ：统计指定列非空值的数量。需要考虑是否有NULL值
  此种方式取决于列是否有索引。如果 列有索引，数据库引擎可能会利用索引进行快速计数。如果没有索引，或者有大量NULL值，性能可能较差，因为需要扫描整个表。

# Spring



---------



| 名字 | 作用             |
| ---- | ---------------- |
| POJO | 数据库实体类型   |
| DTO  | 前端给后端的     |
| VO   | 后端返回给前端的 |
| DAO  | 访问数据库的对象 |



## 注解

- Autowired + Resource

> 2024.4.12
>
> @Autowired是spring特有的注解，默认按照**类型**自动装配，如果有多个相同类型的bean，即一个service有多个impl实现类，则可以通过@Qualifier注解指定注入
>
> @Resource是java带的，默认按照**名称**自动注入，如果没有指定名称，则按照类型

- Repository

> @Repository是属于Spring的注解。它用来标注访问层的类（Dao层），它表示一个仓库，主要用于封装对于数据库的访问。其实现方式与@Component注解相同，只是为了明确类的作用而设立。

- ControllerAdvice + ExceptionHandler

> 因为@ExceptionHandler是局部处理，也就是每个controller都需要加一下，所以可以单独创建一个异常处理类，加上@ControllerAdvice表示是全局处理方式

----------------

## 事务

Spring事务的传播性

| 传播性         | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| REQUIRED(默认) | 当前存在事务A，事务B则会加入A                                |
| SUPPORTS       | 如果当前没有事务，则以非事务运行                             |
| MANDATORY      | 子事务必须加到一个事务中，如果当前没有则报错                 |
| REQUIRES_NEW   | 事务A包括事务B，A回滚，B不会回滚。但B回滚会引起A回滚（需要看外面处不处理异常） |
| NOT_SUPPORTED  | 以非事务运行，如果当前存在事务，则挂起当前事务               |
| NEVER          | 以非事务运行，如果当前存在事务，则抛异常                     |
| NESTED         | 外面回滚，里面也回滚，但里面是独立的                         |



Spring事务失效的场景：

1. 不能用private

2. 用final、static修饰，无法生成代理

3. 这种方式调用updateStauts是用this调用的，解决方法可以用 ` AopContext.currentProxy();`或者注入自己 `UserService`

   ```java
   public class UserService {
       private UserMapper userMapper;
   
       public void add(UserModel userModel){
           userMapper.insert(userModel);
           updateStatus(userModel);
       }
       
       @Transactional
       public void updateStatus(UserModel userModel){
           doSomething()
       }
   
   }
   ```

   

4. 未被Spring管理

5. 多线程导致的错误。在多线程中获取的数据库连接是不一样的，从而是多个事务，因为其实现是用 `ThreadLocal`实现的， `ThreaLocal`是线程间数据隔离的

6. 表不支持引擎。 //TODO 默认引擎是myisam innodb

7. 未开启事务

8. 回滚失效  spring事务回滚是要特定的Exception的 RuntimeException



spring和事务  事务失效  （锁要包裹事务，事务不能包裹锁）

```
2024.4.15补
TransactionAspectSupport 类中的 invokeWithinTransaction()方法中
方法结束才会提交事务，那么并发情况下，线程可以乘虚而入
```

 spring bean代理对象 `AopContext.currentProxy`

```
2024.4.15补
动态代理的原理是创建代理对象后使用反射机制来调用原方法，以此可以实现AOP等
事务@Transactional注解本质是用动态代理实现的，如果直接在A方法中调用B，用的则是this，所以会创建不出代理导致失效
```



---------

# Redis

`SpringDataRedis`

提供了` RedisTemplate`统一 `API`来操作  `redis`

`Redis`缓存更新策略：

内存淘汰（`redis`自己的内存淘汰机制）、超时剔除（给缓存添加`TTL`）、主动更新（用业务逻辑、代码去更新）

先更新数据库->再删除缓存

**缓存穿透**：指redis和数据库中都没有数据，返回空，而不断请求

- 解决方法：1.缓存NULL值  2.布隆过滤 3.增强参数的格式

**缓存雪崩**：大量的缓存`key`失效（`TTL`过期）或者 `redis`服务器宕机，导致大量请求直接到达数据库带来的压力

- 解决方法：1.TTL后添加一定随机值 2.redis集群 3.限流降级 4.多级缓存

**缓存击穿**：热点key（高并发访问，业务复杂的key）突然失效

- 解决方案：1.互斥锁 2.逻辑过期

lambda表达式 匿名函数

 `setSql`用法

悲观锁：认为线程安全问题一定会发生，因为在操作数据之前先获得锁，确保线程串行执行

Synchronized、Lock都属于悲观锁

乐观锁：认为线程安全问题不一定会发生，因此不加锁，只是在数据更新时去判断有没有其他线程对数据做了修改

- 版本号法
- CAS法



//TODO aspectjweaver

//TODO nginx负载均衡 conf配置

nginx负载均衡后，集群模式下（分布式），多个JVM有多个锁监视器，导致synchronized失效，也就是需要跨进程的锁



不可重入的Redis分布式锁，即 `set lock thread1 NX EX`

`setnx`的互斥性，`ex`避免死锁

可重入锁JDK下的ReentrantLock  Redisson也是可重入锁 计数实现 且需要重置锁的有效期

Redisson可重入的分布式锁大概原理：

- 可重入：利用hash结构记录线程ID和重入次数
- 可重试：利用信号量和PubSub（订阅）功能实现等待、唤醒、获取锁失败的重试机制
- 超时续约：利用watchDog，每个一段时间`（releaseTime/3)`，重置超市时间

```
ttl表示获取已有锁的过期时间，null表示当前没有已存在的锁，即可以成功获取锁，

leasetime是锁自动释放的ttl

用了订阅机制
```

基于阻塞队列实现秒杀：

BlockingQueue     @PostConstruct注解 初始化后就执行

注意子线程 不能直接取到proxy,得先取，还有不能用UserHolder中取id，需要从当前传递过来的参数中拿



----------

2024.3.13

基于阻塞队列的异步秒杀是放在JVM中实现的（BlockingQueue），会有可能超出JVM内存，并且JVM不能持久化，所以要放到redis中的消息队列去做

Redis消息队列：生产者、消费者模式

- 基于List双向链表

  支持数据持久化

- 基于PubSub（订阅）

  支持多生产、多消费

  不支持数据持久化（如果发出的消息没有被人订阅，则消息直接丢失）

  消息堆积有上限

- 基于Stream，是一种数据类型

---------

2024.3.19

```java
// 1.查询top5的点赞用户 zrange key 0 4
Set<String> topFive = stringRedisTemplate.opsForZSet().range(key, 0, 4);
// 2.解析出其中的用户id
List<Long> ids = topFive.stream().map(Long::valueOf).collect(Collectors.toList());
```

这段Java代码是将一个流（Stream）对象topFive中的元素转换为Long类型后，再收集到一个新的List中。
topFive应该是一个包含字符串形式的数字（如"1", "2", "3"等）的集合或数组，通过.stream()方法将其转换为Stream流对象。
.map(Long::valueOf)操作符用于将流中的每个元素映射为Long类型。Long::valueOf是一个方法引用来调用Long.valueOf(String)方法，该方法可以将字符串转换为Long。
最后，.collect(Collectors.toList())用于将处理后的流内容收集到一个List容器中。
总结：此段代码目的是将包含字符串数字的集合转化为包含Long类型数字的新列表。



----------

2024.3.21

`GEO` redis中的地理位置接口 用于求附近的人/商家等

`UV Unique Visitor` 独立访客量，1天内同一用户访问多次，只记录1次

`PV Page View` 页面访问量或点击量

`Hyperloglog`原理[HyperLogLog 算法的原理讲解以及 Redis 是如何应用它的 - 掘金 (juejin.cn)]



---

2024.3.28 / 2024.4.19补

主从集群：主（master)用来写，从(slave)用来读

master节点宕机就需要哨兵，哨兵 sentinel， 本身也是集群

哨兵的作用：

1. 监控 会不断检查master和slave是否按预期工作
2. 自动故障恢复 如果master故障，哨兵会提升一个slave为master
3. 通知 充当redis客户端的服务，当发现集群故障转移时，会将最新消息推送到redis客户端 ，有点类似于注册中心（因为哨兵模式下，java客户端访问主节点时并不知道故障没，所以需要访问的是哨兵）

服务状态监控基于心跳机制监测服务状态



分片集群 - 高并发使用，有多个master节点

redis会把每一个master节点映射到0~16383共16384个插槽上，利用CRC16 Hash算法

所以本质上和节点无关，因为节点会宕机更换等

--------

2024.4.8

为什么登录验证需要引入redis，由于负载均衡部署多个tomcat时（虽然现在是单tomcat)，session不共享，所以需要引入一个中间件替代session的功能，并且需要恰好支持key、value结构

ThreadLocal不是针对程序的全局变量，还是针对线程的全局变量

由于Spring底层用的是线程池，所以最好生命周期结束时清空threadlocal，防止线程复用，使用到前世的数据

----------

2024.4.11

乐观锁是用在修改的地方的，悲观锁一般用在查询位置

------------

2024.4.17

Redis的两种持久化

RDB和AOF

RDB（redis database backup file) redis数据快照，就是把内存中数据记录到磁盘中

AOF (Append Only File)追加文件。 redis处理的每一个写命令都会记录在AOF文件

-------

# RocketMQ

2024.4.22

RocketMQ采用的是发布订阅模式

主题（Topic)：一类消息的集合，是RocketMQ进行消息订阅的基本单位

代理服务器（Broker Server) ：消息中转角色，负责存储消息、转发消息

名字服务（Name Server)：管理Broker Server，相当于一个管理机构

---------

2024.4.25

//TODO 消息过程幂等

RocketMQ无法避免消息重复，需要在消费端去重

-------

2024.4.28

RoecktMQ启动，bin目录下

```
nohup sh mqbroker -c ../conf/broker.conf > ../broker.log &
```

-------

2024.5.6

消息堆积 ： 生产太快了，一般认为队列消息差值>=10W算堆积，可以增加消费者数量，但消费者数量 <= 队列数量

----

2024.5.9

QPS过高时，LVS/F5 硬件负载均衡

bean的生命周期：

 实例化、属性赋值、

初始化（前PostConstruct /中InitializingBean/后BeanPostProcesser）initMethod、使用、销毁

---------

2024.5.10

这里yml不配置rocketMq的这个会报注入不进去

```yml
producer:
  group: boot-producer-group
```

//TODO  数据库的行锁



# GitHub

`ctrl + K` 打开命令面板

`s`快速搜索



# Docker

docker的最小运行环境没有vi命令等，所以要挂载数据卷当真实的盘上，能双向链接， `var/lib/docker/volumes`

也就是`docker run` 中的 `-v dest:src`  `dest`是挂载的位置，`src`是容器内的位置

> `/mysql`是卷  `./mysql`是宿主机物理盘

创建`mysql`

```
docker run -d \
  --name cloud \
  -p 3306:3306 \
  -e TZ=Asia/Shanghai \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -v /root/mysql/data:/var/lib/mysql \
  -v /root/mysql/conf:/etc/mysql/conf.d \
  -v /root/mysql/init:/docker-entrypoint-initdb.d \
  mysql
```

创建`minio`

```
docker run \
-p 19000:9000 \
-p 9090:9090 \
--net=host \
--name wmminio \
-d --restart=always \
-e "MINIO_ACCESS_KEY=yxm" \
-e "MINIO_SECRET_KEY=yxm980918" \
-v /root/minio/data:/data \
-v /root/minio/config:/root/.minio \
 minio/minio server \
/data --console-address ":9090" -address ":19000"
```

创建`redis`

```
docker run -itd --name wmredis -p 6379:6379 redis
```

创建`naocs`

```
docker run -d \
--name nacos \
-p 8848:8848 \
-p 9848:9848 \
-p 9849:9849 \
--privileged=true \
--restart=always \
-e JVM_XMS=128m \
-e JVM_XMX=128m \
-e MODE=standalone \
-e PREFER_HOST_MODE=hostname \
-e SPRING_DATASOURCE_PLATFORM=mysql \
-e MYSQL_SERVICE_HOST=47.102.128.67 \
-e MYSQL_SERVICE_PORT=3306 \
-e MYSQL_SERVICE_DB_NAME=nacos \
-e MYSQL_SERVICE_USER=root \
-e MYSQL_SERVICE_PASSWORD=123456 \
-v /root/nacos/logs:/home/nacos/logs \
-v /root/nacos/init.d/custom.properties:/etc/nacos/init.d/custom.properties \
-v /root/nacos/data:/home/nacos/data \
nacos/nacos-server

```



# OAuth2

一种安全协议

调试 OAuth 2.0 接口大致就是上面的步骤，总共分三步：

> 第一步，在登录提供方（如微信或谷歌等）的页面上操作授权，获得授权 code；
> 第二步，在客户端里用授权页返回的 code 申请 access_token；
> 第三步，在客户端里用 access_token 请求更多信息。

- RBAC模型



# SSO

- cookie实现（要求域相同）

> cookie是客户端存储数据的工具。客户端向服务器发送请求时，会带上相同domain(域)的cookie，那只要请求过一次登录，将cookie保存在HttpServletResponse中，这样其他相同域就会带上之前的这个cookie，服务端只要控制跳转即可



# 微信开发

 `appid:` wx7c01f92a8aa18010

`appsecret:` 11a6bf42f8ba5ff581ff38f92e3d4e14

# 网络

文章参考

> [一文彻底搞清session、cookie、token的区别 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/631349844)
>
> [三次握手和四次挥手简单理解-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1670465)

---

2024.4.7

[HTTP协议](https://so.csdn.net/so/search?q=HTTP协议&spm=1001.2101.3001.7020)是无状态的（stateless）是指服务器在处理客户端的请求时，不会将请求和先前状态相关联。简单来说，每一个HTTP请求都是独立的，不保存与先前请求相关的信息。如何解决判断两次请求是同一个人，就加上了cookie和session

| 方法    | 特点                                                | 作用                                                         |
| ------- | --------------------------------------------------- | ------------------------------------------------------------ |
| cookie  | 客户端数据，存储小                                  | 通过请求传递，技术比较老，但容易实现，不够安全，跨域不好解决 |
| session | 保存在服务端，基于cookie，将sessionid保存在cookie中 | 耗费服务器资源，因为相当于保存了一个完整的用户信息，但是是内存操作，时间快，空间换时间 |
| token   | 体积小，C/S端都可以保存                             | token是服务端生成但是保存在客户端的，安全，只有用户ID，跨域处理方便。时间换空间，每次都要去数据库查，但是现在有了redis后，可以缓存使用 |
| jwt     | 解决token依赖数据库或者redis的问题                  | `Token`需要查库验证`Token` 是否有效 而`JWT`不用查库，直接在服务端进行校验。因为用户的信息及加密信息在第二部分`Payload`和第三部分签证中已经生成，只要在服务端进行校验就行，并且校验也是`JWT`自己实现的。但是payload中的数据是可逆的，不建议放敏感信息 |

Q：为什么建立连接是三次握手，而关闭连接却是四次挥手呢？

这是因为**服务端在LISTEN状态下，收到建立连接请求的SYN报文后，把ACK和SYN放在一个报文里发送给客户端。而关闭连接是，当收到对方的FIN报文是，仅仅表示对方不再发送数据了，但是还能接收数据**，已方也未必全部数据都发送给对方了，所以已方可以立即close，也可以发送一些数据给对方后，再发送FIN报文给对方来表示同意现在关闭连接，因此，已方ACK和FIN一般都会分开发送。

