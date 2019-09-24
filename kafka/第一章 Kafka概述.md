# 1.1 kafka简介

​	Apache Kafka 是一个快速、可扩展的、高吞吐的、可容错的分布式“发布-订阅”消息系统, 使用 Scala 与 Java 语言编写,能够将消息从一个端点传递到另一个端点,较之传统的消息中间件(例如 ActiveMQ、RabbitMQ),Kafka 具有**高吞吐量**、**内置分区**、**支持消息副本**和**高容错**的特性,非常适合大规模消息处理应用程序。 

​	Kafka 官网: http://kafka.apache.org/ 

kafka学习框架

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-19-082409.jpg)

# 1.2 kafka系统架构

![](https://ws1.sinaimg.cn/large/006tNc79gy1g1v8r1mh2ej30xs0hq4jl.jpg)

# 1.3 应用场景

​	Kafka 的应用场景很多,这里就举几个最常见的场景。

## 1.3.1 用户的活动追踪

​	用户在网站的活动(网页游览,搜索或其他用户的操作信息)发布到不同的话题中心（topic）,这些消息可实时处理,实时监测,也可加载到 Hadoop 或离线处理数据仓库。这是“用户画像”的一种实现方式。 

## 1.3.2 日志聚合

![](https://ws2.sinaimg.cn/large/006tNc79gy1g1v8tum4s3j30u607a75x.jpg)

## 1.3.3 限流消峰

![](https://ws2.sinaimg.cn/large/006tNc79gy1g1v8uipkwcj30sw07o763.jpg)

# 1.4 kafka高吞吐率实现

​	Kafka 与其它 MQ 相比,其==最大的特点就是高吞吐量==。为了增加存储能力,Kafka 将所有的消息都写入到了低速大容的硬盘。按理说,这将导致性能损失,但实际上,kafka 仍可保持超高的吞吐率,性能并未受到影响。其主要采用了如下的方式实现了高吞吐率。

* 顺序读写
  消息数据存放到分区中，存取磁盘上数据存放是连续的，磁盘磁头不用做寻道，以便提高效率。(顺序写磁盘的性能是随机写入的性能的6000倍的提升，媲美内存随机访问的性能，磁盘不再是瓶颈点。)
* 零拷贝
  零拷贝技术，可以有效的减少上下文切换和拷贝次数。
* 批量发送
  往kafka磁盘上发送、写数据是批量的
* 消息压缩

# 1.5 kafka集群搭建

​	在生产环境中为了防止单点问题,Kafka 都是以集群方式出现的。下面要搭建一个 Kafka集群,包含三个 Kafka 主机,即三个 Broker。

## 1.5.1 kafka的下载

![](https://ws3.sinaimg.cn/large/006tNc79gy1g1v94pig4ej31120nagos.jpg)

⚠️：新版本2.2与以往版本差距较大，主要命令有所差异，配置道无啥差异！

## 1.5.2 kafka安装并配置第一台主机

### （1）上传并解压

​	将下载好的 Kafka 压缩包上传至 CentOS 虚拟机,并解压。

```shell
[root@localhost ~]# cd /usr/tools/
[root@localhost tools]# scp -r changbingbing@192.168.36.173:/Users/changbingbing/Downloads/kafka_2.11-2.2.0.tgz ./
[root@localhost tools]# tar -zxvf kafka_2.11-2.2.0.tgz -C /usr/apps/
```

### （2）创建软链接

```shell
[root@localhost apps]# ln -s kafka_2.11-2.2.0/ kafka
[root@localhost apps]# ll
总用量 4
lrwxrwxrwx.  1 root root    17 4月   2 09:50 kafka -> kafka_2.11-2.2.0/
drwxr-xr-x.  6 root root    89 3月  10 03:47 kafka_2.11-2.2.0
[root@localhost apps]#
```

### （3）修改配置文件

​	在kafka安装目录下有一个 config/server.properties 文件,修改该文件。

```
broker，就是kafka，即当前kafka主机；broker.id，即当前主机在kafka集群中唯一标识；
listeners，即kafka之间通讯使用地址，各主机各写各的，默认是localhost,但建议用ip或主机名；
advertised.listeners，这是producer、consumer和broker通讯的地址，可以不配置，默认用listeners的配置值；
log.dirs，日志目录（配置的目录会自动创建，不用自己手动额外创建），消息是以日志的形式存储；
num.partitions，默认的topic分区，命令中可以单独指定；
zookeeper.connect，zk配置；
```

![](https://ws4.sinaimg.cn/large/006tNc79gy1g1v93ze378j312a0osqju.jpg)

![](https://ws4.sinaimg.cn/large/006tNc79gy1g1v95m2314j310g0k84a6.jpg)

![](https://ws4.sinaimg.cn/large/006tNc79gy1g1v95ysnscj310g0be47j.jpg)



##  1.5.3 再克隆两台Kafka

​	以 kafkaOS1 为母机再克隆两台 Kafka 主机。在克隆完毕后,需要修改 server.properties 中的 broker.id、listeners 与 advertised.listeners。

![](https://ws1.sinaimg.cn/large/006tNc79gy1g1v971gwk8j310k0m0qhs.jpg)

![](https://ws2.sinaimg.cn/large/006tNc79gy1g1v97ap695j310i0jwan6.jpg)

```shell
#办公室
# zk:
zookeeper.connect=192.168.36.175:2181

#kafka
broker.id=0
listeners=PLAINTEXT://192.168.36.230:9092

broker.id=1
listeners=PLAINTEXT://192.168.36.189:9092

broker.id=2
listeners=PLAINTEXT://192.168.36.237:9092
```



## 1.5.4 kafka的启动与停止

### （1）启动zookeeper

```shell
[root@#localhost ~]# zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /usr/apps/zk/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

### （2）启动kafka

​	在命令后添加-daemon 参数,可以使 kafka 以守护进程方式启动,即不占用窗口。

```shell
#这即为一台broker
[root@localhost kafka]# bin/kafka-server-start.sh -daemon config/server.properties
```

### （3）停止kafka

```shell
[root@localhost kafka]# bin/kafka-server-stop.sh
```

## 1.5.5 kafka操作

⚠️：运行命令时，就在kafka这个目录下执行，因为kafka命令较长，不易记忆，所以一般都是直接通过官网锁定具体命令，且官网命令默认从kafka目录开始！

### （1）创建topic

```shell
#-replication-factor 1 指定副本因子值(一般设置为broker的数量，相当于做数据备份，一份是leader,其余都是follower)
#--partitions 1 指定分区值(一般设置为broker的整数倍)
[root@localhost kafka]# bin/kafka-topics.sh --create --bootstrap-server 192.168.36.230:9092 --replication-factor 1 --partitions 1 --topic test
```

### （2）查看topic

```shell
[root@localhost kafka]# bin/kafka-topics.sh --list --bootstrap-server 192.168.36.230:9092
```

### （3）发送消息

```shell
#这里是启一个生产者进程，即此刻该主机既是broker又是producer(安装kafka，启动kafka，它就是一台broker；安装kafka，不启动kafka，直接启一个producer/cousumer进程，它就是纯producer/cousumer)
[root@localhost kafka]# bin/kafka-console-producer.sh --broker-list 192.168.36.230:9092 --topic test
```

### （4）消费消息

```shell
#--from-beginning，表示从头开始消费消息；可以不加该参数，表示仅消费consumer订阅开始的后续消息
[root@localhost kafka]# bin/kafka-console-consumer.sh --bootstrap-server 192.168.36.230:9092 --topic test --from-beginning
```

⚠️：别忘了关闭防火墙，以避免Connection to node -1 (/192.168.36.231:9092) could not be established. Broker may not be available. (org.apache.kafka.clients.NetworkClient)问题！！！

### （5）继续生产消费

```shell
[root@localhost kafka]# bin/kafka-console-producer.sh --broker-list 192.168.36.230:9092 --topic test
>上海
>北京
>
```

```shell
[root@localhost kafka]# bin/kafka-console-consumer.sh --bootstrap-server 192.168.36.230:9092 --topic test --from-beginning
上海
北京
```

### （6）删除topic

```shell
[root@localhost kafka]# bin/kafka-topics.sh --delete --bootstrap-server 192.168.36.230:9092 --topic test
```



