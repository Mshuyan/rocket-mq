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
  + lib：依赖库

#### 配置

##### broker

+ `listenPort`：监听端口，默认10911
+ `namesrvAddr`：名称服务器地址
+ `brokerIP1`：当前broker监听的IP，必填，且不能使用`localhost`或`127.0.0.1`
  `brokerIP2`：存在broker主从时，在broker主节点上配置了`brokerIP2`的话,broker从节点会连接主节点配置的`brokerIP2`来同步
+ `brokerName`：服务名称
+ `brokerClusterName`：加入哪个集群
+ `brokerId`：0代表主节点，正整数代表从节点
+ `storePathCommitLog`：commit log 存放目录`$HOME/store/commitlog/`
+ `storePathConsumerQueue`：消费队列目录，默认`$HOME/store/consumequeue/`
+ `mapedFileSizeCommitLog：commit `log 文件大小，默认`1G`

+ ......

##### 内存大小

+ nameserver

  在`runserver.sh`中

+ broker

  在`runbroker.sh`中

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
  $ nohup ./bin/mqbrokersrv &
  ```

### rocketConsole

+ 从如下地址克隆项目

  https://github.com/apache/rocketmq-externals

+ 使用如下命令编译其中的`rocket-console`项目

  ```sh
  $ mvn clean package -Dmaven.test.skip=true
  ```

+ 使用如下命令启动生成的`rocketmq-console-ng-1.0.1.jar`

  ```sh
  $ nohup java -jar \
      -Drocketmq.config.namesrvAddr=192.168.47.140:9876 \
      -Drocketmq.config.isVIPChannel=false \
      -Dserver.port=9896 \
      rocketmq-console-ng-1.0.1.jar &
  ```

+ 访问：http://localhost:9896

  

