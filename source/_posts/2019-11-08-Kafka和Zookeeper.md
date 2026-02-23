---
title: Zookeeper和Kafka
copyright: true
tags:
  - Devops
  - linux
categories: Linux
abbrlink: e1f23b16
date: 2019-11-08 22:58:30
---
2019-11-08-Zookeeper&Kafka
<!--more-->

# Zookeeper
## [Zookeeper官网](http://zookeeper.apache.org/)

### 1. 什么是Zookeeper?
> Zookeeper是开源`分布式`的，为分布式应用`提供协调服务`的Apache项目；  
> Zookeeper 是一个基于`观察者模式`设计的`分布式服务管理框架`，负责`存储和管理`数据，接受观察者的注册，数据状态一旦发生变化，Zookeeper负责通知已经在Zookeeper注册的观察者做出相应的反应；  
> 应用场景：统一命名服务；统一配置管理；统一集群管理；监控服务器动态上下线；软负载均衡；     

### 2. Zookeeper特点
#### 2.1 Zookeeper的设计目标
{% asset_img zkservice.png [zkservice] %}  

+ 一个Leader，多个Follwers；
+ 集群只要有半数以上节点存活，集群就可以正常服务，适合安装奇数台服务器；
+ 全局数据一致，每个Server保存一份相同数据副本；
+ 数据更新`原子性`，要么成功，要么失败；
+ `实时性`：一定时间内，Client能读到最新数据；
+ `顺序性`：来自同一个Client的更新请求按其发送顺序一次执行；

#### 2.2 Zookeeper数据模型和分层名称 
{% asset_img zknamespace.png [zknamespace] %}   

+ 类似于Unix文件系统，每个节点称为ZNode，默认存储1MB的数据，每个节点可以通过其路径唯一表示；

### 3. Zookeeper的使用
#### 3.1 下载地址
[Zookeeper下载地址](http://zookeeper.apache.org/releases.html)  

#### 3.2 本地模式安装部署
+ JDK
  * `vim /etc/profile`
  ```bash
  JAVA_HOME=/opt/jdk1.8
  JRE_HOME=$JAVA_HOME/jre
  CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib
  export PATH=$JAVA_HOME/bin:$PATH
  ```
+ 解压Zookeeper到/opt

#### 3.3 修改配置文件
+ 路径：/zookeepr-version/conf
  * `mv ./zoo_sample.cfg ./zoo.cfg && mkdir zkData`
  * `vim zoo.cfg`
  ```bash
  tickTime=2000 # 通信心跳时间 2s，S-C
  initLimit=10  # 2sx10=20s，初始Leader与Followed通信时限
  syncLimit=5   # LF同步通信时限，5x2s=10s
  clientPort=2181   # 监听客户端连接端口号
  dataDir=/opt/zookeeper-version/zkData # 数据文件目录和数据持久化路径  
  ```

+ 启动zookeeper服务端&客户端
  * `./bin/zkServer.sh start`
  * `jps`  # Java Virtual Machine Process Status Tool
  * `./bin/zkCli.sh -server IP`  # Command line Iteraction  

#### 3.4 测试
| 命令   | 说明                    |
| :----- | :---------------------- |
| ls     | 查看节点                |
| --     | watch, 监听节点变化     |
| create | 创建节点                |
| --     | -s, 自动建立序号        |
| delete | 删除节点                |
| set    | 设置节点值              |
| get    | 获取节点的值            |
| --     | watch, 监听节点值的变化 |
| stat   | 查看节点状态            |

+ 连接到Zookeeper：
  
* `$ bin/zkCli.sh -server 127.0.0.1:2181` # help 查询帮助命令 
  
+ 创建znode
  ```bash
  [zkshell: 9] create /zk_test "my_data"    # 要有相应的值 
  Created /zk_test

  [zkshell: 11] ls /
  [zookeeper, zk_test]

  [zkshell: 12] get /zk_test    # set /zk_test junk     delete /zk_test
  my_data
  cZxid = 5
  ...
  numChildren = 0
  ```

### 4. 分布式安装部署(建议奇数台)
```bash
vim conf/zoo.cfg
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
initLimit=5
syncLimit=2
server.1=zoo1:2888:3888 # zoo1建议为主机名； 
server.2=zoo2:2888:3888 # 2888：该server与集群leader server交换信息的端口；
server.3=zoo3:2888:3888 # 3888选举端口；

# 配置myid 在数据目录下
vim myid
1
```

# Kafka
## [Kafka官网](http://kafka.apache.org/)

### 1. 什么是Kafka？
> Kafka 是一个`分布式`的`基于发布/订阅模式`的`消息队列`(Message Queue)，主要应用于大数据领域的实时计算以及日志采集。

### 2. 什么是Message Queue？
> 消息队列比作是一个`存放消息的容器`，需要使用消息的时候可以取出消息供自己使用，消息队列是`分布式系统`中重要的组件，目前使用较多的消息队列有`ActiveMQ，RabbitMQ，Kafka，RocketMQ`；队列 Queue 是一种`先进先出`的数据结构。 

> 消息队列的模式：  
  >> 点对点模式，消费者`主动`拉取数据，消息收到后消息清除（阅后即焚）；     
  {% asset_img PP.png [点对点模式] %}   
  >> 发布-订阅模式，发布到Topic数据不会清除，分为push / pull；     
  {% asset_img PS.png [发布-订阅模式] %}   

> 优点：  
  >> 1）通过异步处理提高系统性能（削峰、减少响应所需时间）；   
  >> 2）降低系统耦合性，消息队列使利用`发布-订阅`模式工作，消息发送者（`生产者`）发布消息，一个或多个消息接受者（`消费者`）订阅消息；  

### 3. Kafka的4个核心API
+ `Producer API`：允许应用程序发布的记录流至一个或多个Kafka topics。

+ `Consumer API`：允许应用程序订阅一个或多个主题，并处理所产生的记录流。

+ `Streams API`：允许应用程序扮演流处理器，从一个或多个主题消耗的输入流，并产生一个输出流至一个或多个输出的主题，有效地把输入数据流转变输出流。

+ `Connector API`：允许构建和运行可重复使用的生产者或消费者，将Kafka topics连接到现有的应用程序或数据系统。例如，关系数据库的连接器可能会捕获对表的所有更改。

+ 在Kafka中，客户端和服务器之间的通信是通过简单，高性能，与语言无关的`TCP协议`完成的。
  

{% asset_img kafka-apis.png [kafka-apis] %}    

+ topics：主题是被发布记录的一种类别或者订阅名称，Kafka的主题是通常是多用户的，也即一个主题可以有零个，一个或者多个消费者来订阅写入该主题的数据；对于每个主题，Kafka集群都会维护一个分区日志。

{% asset_img Topic.png [Topic] %}  

### 4. kafka架构：
{% asset_img kafka.png [kafka架构] %}  

  * `Producer`：消息生产者，就是向kafka broker发消息的客户端；
  * `Consumer`：消息消费者，向kafka broker取消息的客户端；
  * `Consumer Group （CG）`：kafka用来实现一个topic消息的广播（发给所有的consumer）和单播（发给任意一个consumer）的手段；广播的实现：让每一个Consuner拥有独立的CG；
    + 第一：同一个消费者组里面，只能有一个消费者去消费分区的数据；
    + 第二：同一个消费者组里面是不会重复消费消息的；
    + 第三：同一个消费者组的一个消费者不是以一条一条数据为单元的，是以`分区为单元`，就相当于消费者和分区建立某种socket，进行传输数据，所以，一旦建立这个关系，这个分区的数据只能是由这个消费者消费；
  * `Broker`：一台kafka服务器就是一个broker；一个集群由多个broker组成。一个broker可以容纳多个topic；
  * `Topic`：可以理解为一个队列；
  * `Partition`：为了实现扩展性，数据量大的topic可以分布到多个broker（即服务器）上，一个topic可以分为多个partition，每个partition是一个有序的队列。partition中的每条消息都会被分配一个有序的id（offset）。kafka只保证按一个partition中的顺序将消息发给consumer，不保证一个topic的整体（多个partition间）的顺序；
  * `Offset`：kafka的存储文件都是按照offset.kafka来命名，用offset做名字的好处是方便查找。例如你想找位于2049的位置，只要找到2048.kafka的文件即可。当然the first offset就是00000000000.kafka；
  * Zookeeper：存储Kafka集群信息，实现集群的高可用；

### 5. kafka安装部署：
#### 5.1 [下载地址](http://kafka.apache.org/quickstart)  
> 依赖于Zk  

#### 5.2 安装部署
```bash
tar -xzvf kafka_version.tgz -C /opt/
```

