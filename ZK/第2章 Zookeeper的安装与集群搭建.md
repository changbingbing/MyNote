# 2.1 安装单机Zookeeper

​	对于系统安全性、实时性要求不是很高的系统,为了节约成本,使用单机 Zookeeper 作为协调服务器也是可以的。

## 2.1.1 准备工作

### （1）克隆并配置机器

克隆一台干净的主机,并修改配置。

* 修改主机名：`/etc/hostname`为`zk001`
* 修改网络配置：`/etc/sysconfig/network-scripts/ifcfg-ens33`

### （2）下载Zookeeper安装包

* [官网下载](http://zookeeper.apache.org/index.html)

## 2.1.2 上传安装包

将下载的 Zookeeper 安装包上传到 `zk001` 主机的`/usr/tools` 目录。

## 2.1.3 安装配置ZK

### （1）解压安装包

```shell
tar -xzvf /usr/tools/zookeeper-3.4.13.tar.gz -C /usr/apps
```

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0fbufgwofj31400aujwo.jpg)

### （2）创建软连接

```shell
ln -s /usr/apps/zookeeper-3.4.13/ /usr/apps/zk
```

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0fbyng0rdj313o0ha14m.jpg)

### （3）复制配置文件

​	复制 Zookeeper 安装目录下的 conf 目录中的 zoo_sample.cfg 文件,并命名为 zoo.cfg:

```shell
cp zoo_sample.cfg zoo.cfg
```

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0fc0z6wwyj310a0kk7h7.jpg)

### （4）修改配置文件

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0fc2hwbzyj30vs0l6k43.jpg)

### （5）新建数据存放目录

```shell
mkdir -p /usr/data/zk
```

### （6）注册bin目录

​	由于我们要使用的命令都在 ZK 安装目录下的 bin 目录中,所以在`/etc/profile` 文件中将该 bin 目录注册到系统环境变量 **PATH** 中。 

```shell
export ZK_HOME=/usr/apps/zk
export PATH=$PATH:$ZK_HOME/bin
```

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0fc4g86gbj30ls09wdjh.jpg)

### （7）重新加载profile文件

```shell
source /etc/profile
```

## 2.1.4 操作zk

### （1）开启zk

```shell
zkServer.sh start
```

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0fc831b7hj30x20a8win.jpg)

### （2）查看状态

```shell
zkServer.sh status
```

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0fc94gpafj30x20aaq72.jpg)

### （3）重启zk

```shell
zkServer.sh restart
```

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0fc9wcqykj30wm0ekaix.jpg)

### （4）停止zk

```shell
zkServer.sh stop
```

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0fcazklb9j30w809o0wu.jpg)

# 2.2 搭建Zookeeper集群

​	对于单机 Zookeeper,存在单点故障问题,系统的安全性降低。Zookeeper 在生产场景中一般是以**集群**的形式出现的,而集群中**法定人数主机的数量一般为奇数**。至于为什么一般 为奇数,后面会分析。 

下面要搭建一个由四台 zk 构成的 zk 集群,其中**一台为 Leader,两台 Follower,一台 Observer**。 

## 2.2.1 克隆并配置第一台主机

### （1）克隆并配置主机

克隆前面单机 Zookeeper 主机后,要修改如下配置文件:

* 修改主机名：`/etc/hostname`为`zkos1`

* 修改网络配置：`/etc/sysconfig/network-scripts/ifcfg-ens33`

### （2）修改zoo.cfg

在 zoo.cfg 文件中添加 zk 集群节点列表。

```shell
#server后的1、2、3、4是myid
#IP后的第一个端口号2888是同步端口号，第二个端口号3888是选举端口号
server.1=192.168.36.231:2888:3888
server.2=192.168.36.234:2888:3888
server.3=192.168.36.235:2888:3888
server.4=192.168.36.237:2888:3888:observer
```

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0fcgcbcdcj30xa0pqh27.jpg)

### （3）删除无效数据

​	zk 的运行会在`/usr/data/zookeeper` 目录中生成一些文件与目录,这些文件和目录与当前的 zkServer 具有绑定关系。由于 **zkos1** 主机复制于原单机 Zookeeper,所以会将其`/usr/data/zookeeper` 目录中生成文件与目录一起复制来,这些数据对于 **zkos1** 来说就是无效数据,可以将其全部删除。

```shell
rm -rf /usr/data/zookeeper/*
```

### （4）创建maid文件

​	在`/usr/data/zookeeper` 目录中创建表示当前主机编号的 `myid` 文件。该主机编号要与 `zoo.cfg` 文件中设置的编号一致。 

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0fcky4dt7j30ug09ggpb.jpg)

## 2.2.2 克隆并配置另两台主机

克隆并配置另外两台主机**zkos2**、**zkos3**的方式是相同的,下面以 **zkos2** 为例。

### （1）克隆主机

克隆前面 **zkos1** 主机后,要修改如下配置文件:

* 修改主机名:`/etc/hostname`为**zkos2**/**zkos3**
* 修改网络配置:`/etc/sysconfig/network-scripts/ifcfg-ens33`

### （2）修改myid

修改 myid 的值与 zoo.cfg 中指定的主机编号相同2/3。

```shell
vi /usr/data/zookeeper/myid
```

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0fcx0w4dij30ro06u76e.jpg)

## 2.2.3 克隆并配置第四台主机

​	第四台主机即为要作 Observer 的主机,除了要完成以上配置,修改 myid 为 4 外,还需 要修改 `zoo.conf` 文件:添加 `peerType=observer`。用于指定当前 Server 即为 Observer。 

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0fcz7mf7oj30tq0og7jc.jpg)

## 2.2.4 zk集群操作

### （1）自动zk集群

​	使用 `zkServer.sh start` 命令,逐个启动每一个 **Zookeeper** 节点主机,并且每启动一台,就查看一下其状态。 

* 在启动第一台 **zkos1** 时,其状态是如下的错误状态。因为其只有一台主机无法完成 Leader 选举。 
  ![](https://ws2.sinaimg.cn/large/006tKfTcly1g0fg455f7cj30jr058ack.jpg)
* 再启动 **zkos2** 后再查看这两台的状态,发现 **zkos1** 为 **Follower**,**zkos2** 为 **Leader**。
  ![](https://ws3.sinaimg.cn/large/006tKfTcly1g0fg5ykl0aj30fi063404.jpg)
* 再启动 **zkos3**,查看其状态,则为 **Follower**。
  ![](https://ws3.sinaimg.cn/large/006tKfTcly1g0fg6mc0y7j30fp05u0ue.jpg)
* 再启动 **zkos4** 后查看其状态,则为 **Observer**。
  ![](https://ws1.sinaimg.cn/large/006tKfTcly1g0fg792zzpj30fg05u0uk.jpg) 



### （2）Leader重新选举

* 将当前的 **Leader** 主机 **zkos2** 停机,模拟 **Leader** 宕机的情况。
  ![](https://ws2.sinaimg.cn/large/006tKfTcly1g0fg8drtvvj30on08u784.jpg)
* 再次查看未停机的 **zkServer**,发现其中有一台已由原来的 **Follower** 变为了 **Leader**,而这
  台一定不会是 **zkos4**。因为其只能是 **Observer**。
  ![](https://ws2.sinaimg.cn/large/006tKfTcly1g0fg95l4gpj30pe08vn0t.jpg)

# 2.3 高可用集群容灾

## 2.3.1 服务器数量的奇数与偶数

​	对于 zk 集群中所包含服务器的数量存在一个误区:为了防止出现赞同票与反对票各占一半的问题,必须要将服务器数量部署为奇数(不包含 Observer)。其实,**部署奇数台服务器确要比偶数台好,但不是为了防止投票平均现象的,因为投票平均,探案无法通过,则该提案全被重提的**。 

​	前面我们说过,无论是写操作投票,还是 Leader 选举投票,都必须过半才能通过,也 就是说若出现超过半数的主机宕机,则投票永远无法通过。基于该理论,由 5 台主机构成的 集群,最多只允许 2 台宕机。而由 6 台构成的集群,其最多也只允许 2 台宕机。即,6 台与 5 台的容灾能力是相同的。基于此容灾能力的原因,建议**使用奇数台主机构成集群,以避免资源浪费**。 

## 2.3.2 容灾设计方案

​	对于一个高可用的系统,除了要设置多台主机部署为一个集群避免单点问题外,还需要 考虑将集群部署在多个机房、多个楼宇。对于多个机房、楼宇中集群也是不能随意部署的, 下面就多个机房的部署进行分析。 

​	在多机房部署设计中,要充分考虑“**过半原则**”,也就是说,尽量**要确保 zk 集群中有过半的机器能够正常运行**。 

### （1）三机房部署

​	在生产环境下,**三机房部署是最常见的、容灾性最好**的部署方案。 

​	假定 zk 集群中机器总数(不含 Observer)为 N,三个机房中部署的机器数量分别为 N1、 N2、N3。 

#### A、N1的值

​	`N1 = (N - 1)/ 2`,例如 N 为 15,则 N1 的值为 7。即要保证第一机房中具有刚不到半数的主机。

#### B、N2的值

​	N2 的值是一个取值范围,`1 ~ (N – N1)/ 2`。仍以 N 为 15 为例,N2 的取值范围为 1~4 台。也就是说,N2 的取值只要不超 4 台即可。 

#### C、N3的值

​	`N3 = N – N1 – N2`。若 N 为 15,N2 为 4,则 N3 为 4。

​	为什么要这样设计呢?这样做可以保证,三个机房中若有一个机房断电或断网,其它两个机房中的机器仍可以正常对外提供服务。当然,若两个机房出现了问题,那么整个集群就瘫痪了。这种情况出现的概率要远低于一个机房出问题的情况。

### （2）双机房部署

​	zk 官网没有给出较好的双机房部署的容灾方案。只能是让其中一个机房占有超过半数的主机,使其做为主机房,而另一机房少于半数。当然,若主机房出现问题,则整个集群会瘫痪。

# 2.4 扩容与缩容

​	水平扩容对于提高系统服务能力,是一种非常重要的方式。但 zk 对于水平扩容与缩容做的并不完美,<u>主机数量的变化需要修改配置文件后整个集群进行重启</u>。集群重启的方式有两种:

## 2.4.1 整体重启

​	整体重启指将整个集群停止,然后更新所有主机的配置后再次重启集群。该方式会使集群停止对外服务,所以该方式**慎用**。

## 2.4.2 部分重启

​	部分重启指每次重启一小部分主机(不能多于半数,生产环境下一般为 1/3)。该方式不影响系统对外服务。