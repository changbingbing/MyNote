# 2.1 kafka基本原理

​	对于 Kafka 基本原理的介绍,可以通过对以下基本概念的介绍进行。

## 2.1.1 Topic

​	**主题**（分类）：在 Kafka 中,使用一个类别属性来划分消息的所属类,划分消息的这个类称为 topic。topic 相当于消息的分类标签,是一个**逻辑概念**。**一个 topic 的 partition 数量是 broker 的整数倍**。

## 2.1.2 Partition

​	**分区**（目录、文件夹）：topic 中的消息被分割为一个或多个 partition,其是一个物理概念,对应到系统上就是一个或若干个目录。

* 注意⚠️：一个Partition只隶属一个topic

* 在磁盘上存放数据是连续的，segment 文件之间存放也是连续的，segment 文件内部数据存放也是连续的；

* 一个topic设置Partition规则：Partition数量＝n*Broker

* 多个Partition无法保证生产消息和消费消息顺序保持一致，因此根据实际经验，为了保证生产消息和消费消息顺序保持一致，一般Partition数量设置1个；

* Partition大小本身没有限制，只和当前硬盘有关；

  ```shell
  问题：
  一个topic中的消息被分割为一个或多个partition（物理概念），分布于多个broker上！如此，自然会有一个问题，无法保证生产消息、消费消息的有序性；
  1、根据实际经验，若要保证生产、消费有序性，则partition数量一般设置为1，分布在一个broker上，对于这种情况而已，其它的broker对于该topic而已就是多余的喽？
  2、是否可以根据具体消息的消费有序性需求来专门设置topic的partition呢？(不过，从一个Partition只隶属一个topic这个角度来看，应该可以专门设置吧)
  ```

  ![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1n8mfmx2xj30rg0f075x.jpg)

## 2.1.3 segment

​	**段**（文件）：将 partition 进一步细分为了若干的 segment,每个 segment 文件的大小相等。

​	理论上有Partition就可以了，干嘛还要segment呢，主要是为了方便管理！因为Partition可能存放的数据量很大，而每个segment大小相等，大小可以设置。

## 2.1.4 Broker

​	**服务器节点**：Kafka 集群包含一个或多个服务器,每个服务器**节点**称为一个 broker。

N 个 partition,M 个 broker：

* 若 N>＝M,且 N 是 M 的整数倍,每个 broker 会平均存储 partition。

* 若 N>M,但 N 不是 M 的整数倍,此时会出现每个 broker 上分配的 partition 数量不一样的情况。

* 若 N<M,则会出现有的 broker 中是没有分配 partition 的情况。

  ```shell
  疑问：partition分区是存数据的，那么如果出现broker中没有分配partition的话，那这个broker还有啥用呢？
  回答：注意，partition是分配在topic维度上的，broker中没有分配partition是说个别topic在这个broker上没有分配partition，而不是说所有的topic在这个broker上都没分配partition。
  ```

注意：每个broker上分几个Partition是系统根据总数量自动分配的。Partition的数量可以自己设置，所以要避免N!=n*M；

## 2.1.5 Producer

​	**生产者**：即消息的发布者,其会将某 topic 的消息发布到相应的 partition 中。

## 2.1.6 Consumer

​	**消费者**：可以从 broker 中读取消息。一个消费者可以消费多个 topic 的消息。但**对于某一个 topic 的消息,其只会消费同一个 partition 中的消息，不能消费多个partition**。

## 2.1.7 Replicas of partition

​	**分区副本**：副本是一个分区的备份,是为了防止消息丢失而创建的分区的备份。

## 2.1.8 Partition Leader

​	每个 partition 有多个副本,其中有且仅有一个作为 Leader,Leader 是当前负责消息读写的 partition。即所有读写操作只能发生于 Leader 分区上。

## 2.1.9 Partition Follower

​	所有 Follower 都需要从 Leader 同步消息,Follower 与 Leader 始终保持消息同步。这些Follower 都会保存在由 Leader 负责维护的 ISR 列表中。

## 2.1.10 ISR

* ISR,In-Sync Replicas,是指**副本同步列表**。由Partition Leader维护，存储在zk中，Follower同步超时或Follower挂了，那么leader会将Follower从ISR列表中移除至OSR！
* OSR,Outof-Sync Replicas,
* AR,Assigned Replicas
* AR = ISR + OSR

## 2.1.11 Partition offset

​	**分区偏移量**：当 consumer 从 partition 中消费了若干消息后,consumer 会将这些消费的消息中最大的 offset 提交给 broker,表示当前 partition 已经消费到了该 offset 所标识的消息。

​	用来标记哪个消息一经被消费过，防止重复消费。

## 2.1.12 Broker Controller

​	Kafka 集群的多个 broker 中,有一个会被选举为 controller,负责管理整个集群中 partition和 replicas 的状态。

## 2.1.13 HW与LEO

​	**HW**：HighWatermark,高水位,表示 Consumer 可以消费到的最高 partition 偏移量。HW 保证了 Kafka 集群中消息的一致性。 

​	**LEO**：Log End Offset,日志最后消息的偏移量。消息在 Kafka 中是被写入到日志文件中的,这是当前最后一个消息在 Partition 中的偏移量。 

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-19-022419.png)

​	对于 leader 新写入的消息,consumer 是不能立刻消费的。leader 会等待该消息被所有 ISR 中的 partition follower 同步后才会更新 HW,将 HW 写入到 ISR 中,此时消息才能被 consumer 消费。

​	每个partiton都有自己的LEO，HW是大家的(短板效应)

## 2.1.14 Zookeeper

​	Zookeeper 负责维护和协调 broker。当然,zk 还负责 Broker Controller 的选举。 

​	需要注意,从 Kafka 0.9 开始 offset 的管理与保存机制发生了很大的变化——zookeeper 中不再保存和管理 offset 了。

## 2.1.15 Consumer Group

​	**consumer group** 是 kafka 提供的可扩展且具有容错性的消费者机制。组内可以有多个消费者,它们共享一个公共的 ID,即 group ID。组内的所有消费者协调在一起来消费订阅主题的所有分区。 

​	Kafka 保证同一个 consumer group 中只有一个 consumer 会消费某条消息,实际上,Kafka 保证的是稳定状态下每一个 consumer 实例只会消费某一个或多个特定的数据,而某个 partition 的数据只会被某一个特定的 consumer 实例所消费。 

## 2.1.16 Coordinator

​	Coordinator 一般指的是运行在每个 broker 上的 group Coordinator,用于管理 Consumer。Group 中的各个成员,主要用于 offset 位移管理和 Rebalance,可以同时管理多个消费者组。

## 2.1.17 Rebalance

​	当消费者组中消费者数量发生变化,或 Topic 中的 partition 数量发生了变化时,partition的所有权会在消费者间转移,即 partition 会重新分配,这个过程称为再均衡 Rebalance。

* 消费者组中新增消费者
* 消费者关闭或宕机或取消订阅
* Topic 新增/删除 partition

## 2.1.18 offset commit

​	Consumer 在消费过消息后需要将其消费的消息的 offset 提交给 broker,以让 broker 记录下哪些消息是消费过的。记录已消费过的 offset 值有什么用呢?除了用于标识哪些消息将来要被删除外,还有一个很重要的作用:**在发生再均衡时不会引发消息的丢失或重复消费**。

# 2.2 kafka工作原理与过程

## 2.2.1 消息路由

消息要写入到哪个 Partition 并不是随机的,而是有路由策略的（api方式，命令中没有）：

1. 若指定了 partition,则直接写入到指定的 partition; 
2. 若未指定 partition 但指定了 key,则通过对 key 的 hash 值与 partition 数量**取模**,该取模结果就是要选出的 partition 索引; Record:key-value(key的存在就是为了路由，value就是真正要发的消息集) 
3. 若 partition 和 key 都未指定,则使用**轮询算法**选出一个 partition。 

## 2.2.2 消息写入算法

消息发送者将消息发送给 broker,并形成最终的可供消费者消费的log,是一个比较复杂的过程。 

1. producer 先从 zookeeper 中找到该 partition 的 leader 
2. producer 将消息发送给该 leader 
3. leader 将消息写入本地 log 
4. ISR 中的 followers 从 leader 中 pull 消息,写入本地 log 后向 leader 发送 ACK 
5. leader 收到所有 ISR 中的 followers 的 ACK 后,增加 HW 并向 producer 发送 ACK 

## 2.2.3 HW截断机制(⚠️ )

​	==HW 的截断机制可以保证数据一致性，但不能保证数据不丢失(只是尽量减少数据丢失)==

​	如果 partition leader 接收到了新的消息,其它 ISR 中 Follower 正在同步过程中,还未同步完毕时 leader 宕机。此时就需要选举出新的 leader。若没有 HW 截断机制,将会导致 partition中 offset 数据的不一致。为了避免这种情况的发生,引入了 HW 的截断机制（这样会导致数据丢失，但比起数据的不一致问题，可以接受的）。

​	==HW截断机制比较重要，原理演示==分析如下：

​	比如有3台主机：Leader A、B、C，里面暂时有1、2、3 三个消息(此时HW和LEO都在消息3这个位置)
![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-19-023137.png)

1. 正常场景：producer生产消息4、5、6

   * 首先进入LeaderA，此时LeaderA的LEO在消息6这个位置(每个主机都有自己的LEO，HW是集群Leader＋Follower共有的)；
     ![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-19-024522.png)
   * 接下来B、C开始从Leader A同步消息，B同步了4、5消息，此时B的LEO在消息5这个位置；C所在主机性能有限，暂时只同步了4，此时C的LEO在消息4这个位置；此时，B、C都同步到了消息4，因此HW就涨上去到了消息4这个位置，这就意味着消息4现在可以被Consumer消费了（消息5、6还不能被消费）； 
     ![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-19-024346.png)
   * 紧接着，B同步了6，C同步了5，发现B、C都同步到了消息5，因此HW又涨上去到了消息5这个位置；
     ![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-19-024810.png)
   * 最后，C虽然慢，但最终也同步了6，此时三台主机LEO都在消息6这个位置，HW也涨至消息6位置了，这个过程就展示了什么是HW，什么是LEO。
     ![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-19-025009.png)

2. 异常场景（以上是正常流程，假设没有截断机制，会发生什么事情呢？）

   * 首先，同上producer生产消息4、5、6，进入Leader A中，B、C同步消息至上面第二步，如图：
     ![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-19-024346.png)

   * 接下来，B、C继续同步，突然Leader A宕机了（A随之进入了OSR队列），此时B、C都在ISR中，所以要选举新的Leader。如何选举呢？这个选举Leader非常简单，ISR是个队列，它是直接从队列中拿取队首元素，不会像ZK那样做很复杂的选举。
     ![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-19-030731.png)
   * 假设拿到的队首元素是B，那么B就是Leader了，同时producer又生产新消息6、7
     ![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-19-031525.png)
   * 接着，C要同步Leader B消息。假设刚好同步了消息5，HW随之涨至消息5位置
     ![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-19-031645.png)
   * C刚好同步了消息5，突然发现A又活了（Broker Controller检测到OSR中的A复活，允许其进入ISR中，成为Follower)
     ![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-19-032316.png)
   * Follower A要从Leader B中同步数据，即同步消息7，此时会发现什么问题？A、B的消息6发生数据不一致了
     ![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-19-032540.png)
   * 因此，在没有HW截断机制的情况下，会引发数据不一致问题

3. 异常场景，有截断截止会如何？还从上面异常场景第4步A复活说起
   ![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-19-033409.png)

   * 主机A复活，截断机制，首先会将A的LEO调整至它宕机那一刻的HW位置（**HW记录在ISR中，ISR记录在ZK中**），即LEO从A的消息6位置挪到消息4的位置，截断也就是说从宕机时的HW位置开始截断，后面的数据A的5、6都不要了
     ![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-19-033854.png)
   * 截断后，然后重新再做同步(将Leader B的5、6、7同步至A)，此时数据肯定都一致！同时，我们也发现A原来的消息6丢失了
     ![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-19-033955.png)
   * 因此，HW截断机制存在数据丢失问题（相比数据不一致问题，数据丢失可以接受)

## 2.2.4 消息发送的可靠性机制

​	生产者向 kafka 发送消息时,可以选择需要的可靠性级别。通过 request.required.acks参数的值进行设置。

### （1）0值

​	**异步发送**。生产者向 kafka 发送消息而**不需要 kafka 反馈成功 ack**。 

存在问题（效率高，但可靠性低，易丢失数据）：

* kafka 根本就没有收到消息
* kafka 的 buffer 满了,要向 partition 写入消息,在还未写入的瞬间,生产者又生产了新的消息并发送了过来,那么,这些消息也就丢失了
* 消息的生产顺序与写入到 kafka 中的顺序不一致 

### （2）1值

​	**同步发送**。生产者发送消息给 kafka,**broker 的 partition leader 在收到消息后马上发送成功 ack(无需等待 ISR 中的 follower 同步)**,生产者收到后知道消息发送成功,然后会再发送消息。如果一直未收到 kafka 的 ack,则认为消息发送失败,会自动重发消息。

​	Leader的突然宕机，也可能存在数据丢失的问题（不过，HW的截断机制本身可以保证数据一致性问题，不保证数据不丢失）；

### （3）-1值

​	**同步发送**。生产者发送消息给 kafka,**kafka 收到消息后要等到 ISR 列表中的所有副本都同步消息完成后,才向生产者发送成功 ack**。如果一直未收到 kafka 的 ack,则认为消息发送失败,会自动重发消息。 

​	存在消息重复的问题（kafka重复接收、Consumer重复消费）。 

​	kafka 引入了唯一标识与去重机制。

## 2.2.5 消费者消费过程解析

生产者将消息发送到 topic 中,消费者即可对其进行消费,其消费过程如下: 

* 消费者订阅指定 topic 的消息 
* broker controller 会为消费者分配 partition,并将该 partitioin 的当前 offset 发送给消费者 
* 当 broker 接收到生产者发送的消息时,broker 会将消息推送给消费者 
* 消费者接收到 broker 推送的消息后对消息进行消费 
* 当消费者消费完该条消息后,消费者会向 broker 发送一个该消息已被消费的反馈 
* 当 broker 接到消费者的反馈后,broker 会更新 partition 中的 offset 
* 以上过程一直重复,直到消费者停止请求消息 
* 消费者可以重置 offset,从而可以灵活消费存储在 broker 上的消息 

## 2.2.6 Partition Leader选举范围

​	当 leader 宕机后 broker controller 会从 ISR 中选一个 follower(队首元素) 成为新的 leader。但若 ISR中的所有副本都宕机怎么办?可以通过 `unclean.leader.election.enable` 的取值来设置 Leader选举的范围。

### （1）false

​	必须等待 ISR 列表中有副本活过来才进行新的选举。该策略可靠性有保证,但可用性低。

### （2）true

​	可以选择任何一个没有宕机的 follower,但该 follower 可能不在 ISR 中。(OSR中的follower是同步超时或挂掉的follower，若选择OSR中的follower为Leader，本身它自身有问题，由于它是Leader，它会认为别的follower有问题，并将他们从ISR列表中移除至OSR，如此便没有保障，可用性降低)

## 2.2.7 重复消费问题及解决方案

最常见的重复消费有两种:

* 同一个 consumer 重复消费：consumer 本身没有消费完消息，所以也没有发送 offset ，重新拉取消费；

* 同一消费者组中不同的 consumer 重复消费：consumer 消费完了消息,也发送了 offset,但由于网络原因 partition 没有收到 offset，partition的offset没有更新，其他的consumer又接着原来的offset继续消费(网络原因，consumer被认为宕机，重新分配rebalance，其它consumer接着原来的offset消费)

  

  无论哪种重复消费,其解决方案都一样,有两种:

* 增加会话超时时限 session.timeout.ms 的值,延长 offset 提交时间; 

* 设置 enable.auto.commit 为 false,将 kafka 自动提交 offset 改为手动提交(其实，代码中通常用的就是手动提交) 

# 2.3 日志查看

## 2.3.1 查看段segment

### （1）segment文件名

​	segment 是一个逻辑概念,其由两类物理文件组成,分别为“.index”文件和“.log”文 件。“.log”文件中存放的是消息,而“.index”文件中存放的是“.log”文件中消息的索引。 

​	这两个文件的文件名是成对出现的,即会出现相同文件名的 log 与 index 文件。文件名由20 位数字字符组成,其要表示一个 64 位长度的数值(2 的 64 次方是一个长度为 20 的数字)。但作为文件名,其数值长度不足 20 位的全部用 0 填补。 

* 第0号segment文件 

  00000000000000000000.index 

  00000000000000000000.log 

* 第 170210 号 segment 文件 

  00000000000000170210.index 

  00000000000000170210.log 

* 第 239430 号 segment 文件 

  00000000000000239430.index 00000000000000239430.log 

### （2）segment文件内容

![](https://ws1.sinaimg.cn/large/006tNc79gy1g1w82a5t8rj30z20ni48g.jpg)

### （3）消息的查找

​	在 partition 中是如何通过 offset 查找到消息的呢?以查找 offset 为 170213 的消息为例进行分析。 

​	首先拿 170213 这个 offset 通过二分法在所有 segment 文件名中查找对应文件。当定位到00000000000000170210.index 文件后,再进行减法运算:170213 – 170210 =3。然后再在该 index 文件中定位到 2 号(编号从 0 开始),查找到其偏移地址为 348。最后在当前 log 文 件中直接定位到 348 地址处即可找到该消息。 

### （4）查看segment

对于 segment 中的 log 文件,不能直接通过 cat 命令查找其内容,而是需要通过 kafka自带的一个工具查看。 

```shell
bin/kafka-run-class.sh kafka.tools.DumpLogSegments --files /tmp/kafka-logs/test-0/00000000000000000000.log --print-data-log
```

该命令中的 kafka-run-class.sh 表示要运行一个指定的代码。DumpLogSegments 就是一个专门用于查看消息日志文件的工具。

![](https://ws1.sinaimg.cn/large/006tNc79gy1g1w88s5tsij310g0cqnby.jpg)



## 2.3.2 查看offset存储目录

### （1）查看存储目录

​	前面我们讲过,消费者会向一个称为“__consumer_offset”的特殊主题发送消息,消息里包含每个分区的偏移量。通过这种方式提交 offset。既然是主题,其就会有对应的分区,这个特殊主题默认包含 50 个分区。这 50 个分区分别存放在所有 broker 中。

![](https://ws4.sinaimg.cn/large/006tNc79gy1g1wcqiwe34j312m0fy16b.jpg)

任意打开一个“__consumer_offsets-数字”目录,均可以看到其中的 segment。

![](https://ws4.sinaimg.cn/large/006tNc79gy1g1wczgo27sj312c07ijxb.jpg)

### （2）offset存放分区计算

​	每一个用户自定义主题的 offset 都会存放在同一个“__consumer_offsets-数字”目录中。那么存放在哪个目录呢?其计算方法是 topic 主题字符串的 hash 值与 50 取模的结果即为“__consumer_offsets-数字”目录的数字。
