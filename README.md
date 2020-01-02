# rocket-mq

## 下载安装

### rocketMq

#### 下载

+ 地址：https://rocketmq.apache.org/dowloading/releases/
+ 目录介绍
  + benchark：测试脚本
  + bin：启动脚本
    + mqnamesrv：名称服务器启动脚本
    + mqbroker：消息队列服务器启动脚本
    + mqshutdown：停止服务器脚本
  + conf：配置文件
    + 4个目录：broker配置文件示例
    + broker.conf：broker配置文件示例
    + plain_acl.yml：用户权限配置
    + logback-xxx.xml：日志配置
  + lib：依赖库

#### 启动

+ 配置`JAVA_HOME`环境变量

  ```sh
  export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64/"
  ```

+ 启动`nameserver`

  ```sh
  $ nohup ./bin/mqnamesrv &
  ```

+ 配置broker

  增加如下配置

  ```conf
  listenPort=9886
  namesrvAddr=192.168.47.140:9876
  brokerIP1=192.168.47.140
  ```

+ 启动`broker`

  ```sh
  $ nohup ./bin/mqbroker -c ./conf/broker.conf &
  ```

#### 停止

+ 停止namesrv

  ```sh
  $ ./bin/mqshutdown namesrv
  ```

+ 停止broker

  ```sh
  $ ./bin/mqshutdown broker
  ```

### rocketConsole

+ 从如下地址克隆项目

  https://github.com/apache/rocketmq-externals

+ 使用如下命令编译其中的`rocket-console`项目，生成`rocketmq-console-ng-1.0.1.jar`

  ```sh
  $ mvn clean package -Dmaven.test.skip=true
  ```

+ 拷贝出`application.properties`文件，与`rocketmq-console-ng-1.0.1.jar`放在同1个目录下

  修改如下配置项：

  + `server.port`
  + `rocketmq.config.namesrvAddr`
  + `rocketmq.config.dataPath`
  + `rocketmq.config.loginRequired`

  ```properties
  server.contextPath=
  server.port=9896
  
  # SSL 设置
  #server.ssl.key-store=classpath:rmqcngkeystore.jks
  #server.ssl.key-store-password=rocketmq
  #server.ssl.keyStoreType=PKCS12
  #server.ssl.keyAlias=rmqcngkey
  
  #spring.application.index=true
  spring.application.name=rocketmq-console
  spring.http.encoding.charset=UTF-8
  spring.http.encoding.enabled=true
  spring.http.encoding.force=true
  logging.config=classpath:logback.xml
  # 名称服务器地址
  rocketmq.config.namesrvAddr=192.168.47.140:9876
  # 默认true
  rocketmq.config.isVIPChannel=
  # 控制台的文件存储路径
  rocketmq.config.dataPath=/tmp/rocketmq-console/data
  # 默认true
  rocketmq.config.enableDashBoardCollect=true
  # 消息轨迹topic名
  rocketmq.config.msgTrackTopicName=
  rocketmq.config.ticketKey=ticket
  
  # 控制台是否启动登录功能，默认false，如果启用了，
  # 必须创建 ${rocketmq.config.dataPath}/users.properties文件，并配置用户名密码
  rocketmq.config.loginRequired=true
  ```

+ 创建`${rocketmq.config.dataPath}/users.properties`文件，

  ```properties
  # admin:				用户名
  # qsc123456:		密码
  # 1：						用户类型：0：普通用户，1：管理员
  admin=qsc123456,1
  ```
  
+ 使用如下命令启动生成的`rocketmq-console-ng-1.0.1.jar`

  ```sh
  $ nohup java -jar rocketmq-console-ng-1.0.1.jar &
  ```
  
+ 访问：http://localhost:9896

## 概念

### 刷盘

刷盘指的是将内存中的消息落地到磁盘中

+ 同步刷盘

  生产者生产消息后，broker直接将消息落地，然后返回响应

+ 异步刷盘

  生产者生产消息后，broker先将消息放在内存中，直接返回响应，后续再将消息落地

### 重试

`rocketmq`自身具备重试机制，1条消息消费失败后自己会进行重试

TODO:具体了解下机制

### 死信

1条消息多次未消费成功，就会进入死信队列，后续可以手动继续投递

### 消息有序

+ 如何保证消息有序

  要保证消息有序，需要保证2点：

  + 生产者将多个需要有序消息按照顺序发送到同1个queue中
  + 对于同1个queue，只由1个消费者进行消费

+ 普通消息

  + 生产时不保证消息的顺序，轮训选择1个queue发送消息

  + 这样的发送时，每条消息都不能保证消费顺序

+ 分区顺序消息

  + 生产者将多个需要有序消息按照顺序发送到同1个queue中
  + 对于同1个queue，只由1个消费者进行消费
  + 这样可以保证这几个消息之间是有序的，而与其他消息之间是无序的

+ 全局顺序消息

  将所有消息按照顺序发送到同个queue中

  只有1个消费者消费这些消息

### 生产模式

+ 同步消息
  + 需要等待服务器响应，才能发送下一条
  + 一般用于重要的场景

+ 异步消息
  + 不需要等待服务器响应，直接发送下一条，但是注册了回调函数，用于处理响应
  + 一般用于网络链路比较耗时的场景

+ 单向消息
  + 不需要等待服务器响应，直接发送下一条，也没有回调函数处理
  + 一般用于日志手机等场景

### 消费者类型

+ pull

  消费者主动拉取消息

+ push

  消费者与broker建立长链接，broker发现消息后，主动发送消息给消费者

### 消费模式

+ 集群模式
  + 默认
  + 1条消息只能被消费组中1个消费者消费
+ 广播模式
  + 1条消息会被消费组中每个消费者消费1次

### topic、queue

+ topic就是多个queue的集合，每个队列之间没有交集，各走各的

  topic与queue的关系，可以理解为网线与网线内部的8根铜丝的关系，铜丝之间相互没有任何干扰，但是同属于1个网线

+ 生产者发送消息，都是向topic中发送，queue的存在是为了适应高并发场景

+ 创建topic时，读写队列数要一致

### 生产者、生产组

+ 生产者指的是生产消息的1个应用实例
+ 生产组指的是生产相同消息的一组生产者的集合
+ 用一个生产组中的各个生产者生产的消息应该是一致的
+ 生产者生产的消息只能发送到1个队列中，轮训决定发送到哪个队列
+ 1个生产组可以向多个topic发送消息，1个topic可以接收来自不同生产组的消息
+ 实际应用中，一般不指定生产组

### 消费者、消费组

+ 消费者指的是消费消息的1个应用实例

+ 消费组指的是消费相同消息的一组消费者的集合

+ 1个消费组内的所有消费者，要具备完全相同的订阅关系，消费相同的数据

+ 1个消费组可以订阅多个topic，1个topic也可以被多个消费者订阅

  但是感觉，常规情况下，消费组与topic也应该是一对一关系，只有1个topic需要以广播模式被按照不同的逻辑消费时，才会出现一对多的关系

+ **同1个消费组内**，1个队列只能被1个消费者订阅，1个消费者可以订阅多个队列

### 消息过滤

+ tag过滤
  + 生产者生产消息时，可以为消息指定tag（多tag间用`||`分割）
  + 消费者在订阅消息时，也可以指定tag（多tag间用`||`分割），当要消费的消息不包含指定的tag时，则丢弃

+ SQL92过滤
  + 使用`SQL92`表达式进行过滤
  + （以后再研究）

### 消息轨迹

+ 默认启动

  ```properties
  spring.cloud.stream.rocketmq.binder.enableMsgTrace=true
  ```

+ 默认消息轨迹topic：

  ```properties
  spring.cloud.stream.rocketmq.binder.customizedTraceTopic=RMQ_SYS_TRACE_TOPIC
  ```

+ 必须指定了账号密码时，轨迹功能才生效

## 配置

### broker

+ `listenPort`：监听端口，默认10911
+ `namesrvAddr`：名称服务器地址
+ `brokerIP1`：当前broker监听的IP，必填，且不能使用`localhost`或`127.0.0.1`
  `brokerIP2`：存在broker主从时，在broker主节点上配置了`brokerIP2`的话,broker从节点会连接主节点配置的`brokerIP2`来同步
+ `brokerName`：服务名称
+ `brokerClusterName`：加入哪个集群，
+ `brokerId`：0代表主节点，正整数代表从节点
+ `storePathCommitLog`：commit log 存放目录`$HOME/store/commitlog/`
+ `storePathConsumerQueue`：消费队列目录，默认`$HOME/store/consumequeue/`
+ `mapedFileSizeCommitLog`：commit log 文件大小（单位B），默认`1G`
+ `autoCreateTopicEnable`：是否允许broker自动创建topic
+ `defaultTopicQueueNums`：自动创建topic时默认队列数
+ `autoCreateSubscriptionGroup`：是否允许broker自动创建订阅组
+ `deleteWhen`：删除文件时间点，4表示凌晨4点
+ `fileReservedTime`：文件保留小时数，默认48小时
+ `mapedFileSizeComsumeQueue`：每个文件可以存多少条数据（单位条），默认30W
+ `brokerRole`：broker服务角色，可选值：`SYNC_MASTER`/`ASYNC_MASTER`/`SLVAE`，默认`ASYNC_MASTER`
+ `flushDiskType`：刷盘策略，可选值`SYNC_FLUSH`、`ASYNC_FLUSH`（默认值）
+ `aclEnable`：是否启动用户认证，默认false

### 内存大小

+ nameserver

  在`runserver.sh`中

+ broker

  在`runbroker.sh`中

### 日志路径

`conf`目录下的`logback_xxx.xml`是日志配置文件，需要修改日志存放路径，可以将这3个文件中对应的`${user.home}$`替换为你想要存放的路径

## ACL

+ broker.conf

  ```conf
  aclEnable = true
  ```

+ plain_acl.yml

  + 账号密码值使用数字加字母，使用`!`时报错了，且长度至少7位
  + broker启用了ACL时，必须将`rocketmq-concole`的ip加入白名单，否则控制台就失效了，目前`rocket-console`还不支持`acl`功能

  ```yml
  globalWhiteRemoteAddresses:
  - 192.168.1.*
  - 127.0.0.1
  
  accounts:
  - accessKey: rocketmq0
    secretKey: qsc12345678
    whiteRemoteAddress:
    admin: false
    defaultTopicPerm: DENY
    defaultGroupPerm: SUB
  
  - accessKey: rootroot0
    secretKey: qsc123456
    whiteRemoteAddress: 
    admin: true
  ```

+ 业务代码中需要指定账号密码

  ```yml
  spring:
      stream:
        rocketmq:
          binder:
            name-server: 192.168.1.204:9876
            access-key: rootroot0
            secret-key: qsc123456
  ```

## 集成

### springboot方式使用

#### 引入

+ 依赖

  ```xml
  <dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>2.0.4</version>
  </dependency>
  ```

+ application.yml

  ```yml
  rocketmq:
    name-server: 192.168.1.204:9876
    producer:
      access-key: admin00
      secret-key: qsc123456
      group: ${spring.application.name}
  ```

#### 生产消息

> 使用`RocketMqTemplete`发送

+ `send`：

  内部调用了`syncSend`

+ `convertAndSend`：

  先进行了`convert`操作，随后调用了`send`，其实最后调用的还是`syncSend`

  `send`只能发送`Message`对象，`convert`是将要发送的对象转为`Message`

+ `syncSend`

  同步普通消息

+ `syncSendOrderly`

  同步顺序消息

  对于需要保证顺序的多个消息，通过指定相同的hash值，保证分配到同1个queue中

+ `asyncSend`

  异步普通消息

  注册回调函数处理返回响应

+ `asyncSendOrderly`

  异步顺序消息

  注册回调函数处理返回响应

  对于需要保证顺序的多个消息，通过指定相同的hash值，保证分配到同1个queue中

+ `sendOneWay`

  单向普通消息

+ `sendOneWayOrderly`

  单向顺序消息

  对于需要保证顺序的多个消息，通过指定相同的hash值，保证分配到同1个queue中

+ `createAndStartTransactionMQProducer`

+ `sendMessageInTransaction`

+ `removeTransactionMQProducer`

+ 定时延时消息

  + 目前只支持固定精度级别的定时消息，服务器按照1-N定义了如下级别： “1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h”；

  + 若要发送定时消息，在应用层初始化Message消息对象之后，调用Message.setDelayTimeLevel(int level)方法来设置延迟级别，按照序列取相应的延迟级别，例如level=2，则延迟为5s

#### 消费消息





### stream方式使用

> 学习的时候感觉stream方式使用起来并不那么灵活，决定先用springboot方式使用，stream方式以后在研究
>
> 资料：
>
> + [spring-cloud-alibaba/RocketMQ](https://github.com/alibaba/spring-cloud-alibaba/wiki/RocketMQ) 

+ 依赖

  ```xml
  <dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rocketmq</artifactId>
  </dependency>
  ```

+ application.yml

  ```yml
  spring:
    cloud:
      stream:
        rocketmq:
          binder:
            name-server: 192.168.1.204:9876
            access-key: admin00
            secret-key: qsc123456
        bindings:
          search-job-output:
            destination: search-job
          search-job-input:
            destination: search-job
            group: search-job-consumer
  ```

  配置说明：

  + biddings：用于配置多个`channel`（通道），每个通道是且只能是1个topic的生产者**或**消费者
    + ${channel-name}：`channel`名称
      + destination：对应的topic名称
      + group：如果想把这个`channel`作为消费者用，必须配置`group`属性，该属性也仅对消费者有效



