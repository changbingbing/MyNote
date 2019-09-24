# 4.1 重要概念

## 4.1.1 数据模型Znode

![](https://ws3.sinaimg.cn/large/006tKfTcly1g0kr768gyrj311g0swag8.jpg)

​	**zk** 数据存储结构与标准的 **Unit** 文件系统非常相似,都是**在根节点下挂很多多级数据节点**。<u>zk 中没有引入传统文件系统中目录与文件的概念,而是使用了称为 **znode 的数据节点**概念</u>。**znode** 是 **zk** 中数据的最小单元,每个**znode** 上都可以保存数据,同时还可以挂载子节点,形成一个树形化命名空间。

### （1）节点类型

​	每个 **znode** 根据节点类型的不同,具有不同的生命周期。 

* 持久节点 ：数量最多且一直存在的节点，直到被删除
* 持久顺序节点 ：
  * 有创建顺序性的节点；
  * 与持久节点区别主要体现在名称上，名称上带有一串数字，这个数字由父节点维护；
* 临时节点 ：
  * 临时性的节点；
  * 生命周期与会话绑定的，会话消失了，临时节点也会干掉了；
  * ⚠️**临时节点只能做叶子节点（叶子节点不能有子节点）**，即：
    * 临时节点肯定是叶子节点，但叶子节点未必是临时节点，也可能是持久节点
* 临时顺序节点 ：有创建顺序性的临时节点

### （2）节点状态

​	每个 **znode** 节点,除了其要存放的数据以外,还有描述其自身状态的元数据信息。这些元数据信息通过后续客户端 `get` 命令可以查看到。这些元数据信息分别是: 

* **cZxid**：`Created Zxid`,表示当前 znode 被创建时的事务 ID 
* **ctime**：`Created Time`,表示当前 znode 被创建的时间 
* **mZxid**：`Modified Zxid`,表示当前 znode 最后一次被修改时的事务 ID 
* **mtime**：`Modified Time`,表示当前 znode 最后一次被修改时的时间 
* **pZxid**：表示当前 znode 的<u>子节点列表</u>最后一次被修改时的事务 ID。**注意**：<u>只能是其子节点列表变更了才会引起 pZxid 的变更,子节点内容的修改不会影响 pZxid</u>。 
* **cversion**：`Children Version`,表示子节点的版本号。该版本号用于充当乐观锁。 
* **dataVersion**：表示当前 znode 数据的版本号。该版本号用于充当乐观锁。 
* **aclVersion**：表示当前 znode 的权限 ACL 的版本号。该版本号用于充当乐观锁。 
* **ephemeralOwner**：若当前 znode 是持久节点,则其值为 0;若为临时节点,则其值为创建该节点的会话的 `SessionID`。当会话消失后,会根据 `SessionID` 来查找与该会话相关的临时节点进行删除。 
* **dataLength**：当前 znode 中存放的数据的长度。 
* **numChildren**：当前 znode 所包含的子节点的个数。 

## 4.1.2 会话

​	**会话**是 zk 中最重要的概念之一,客户端与服务端之间的任何交互操作都与会话相关。 

​	ZooKeeper 客户端启动时,首先会与 zk 服务器建立一个 `TCP` 长连接,从第一次连接建立开始,客户端会话的生命周期也开始了。通过这个长连接,客户端能够通过**心跳检测**保持与服务器的有效会话,也能够向 ZooKeeper 服务器发送请求并接受响应,同时还能通过该连接接收来自服务器的 `WatcherEvent` 通知。 

### （1）会话状态

​	会话的状态有三种: 

* **connecting** 
  * 从客户端创建zk对象开始，客户端就进入`connection`状态；
  * 然后客户端从服务器列表（服务器列表存在客户端配置中）逐个尝试IP连接，因为是集群，所以只要连上一个就OK了
* **connected**
  * 客户端一旦连上服务列表中一个IP就进入`connected`状态
* **close**：会话关闭（会话超时、权限验证失败、客户端主动退出等）

### （2）重连

​	客户端与服务端的长连接如果失效,则客户端会自动进行**反复重连**。而这个失连后的状态有三种：

* **连接丢失 `CONNECTION_LOSS`**：网络抖动等原因导致连接断开（从`connected`一旦断了就进入`CONNECTION_LOSS`状态），随即客户端会再次从服务列表中逐个尝试IP连接。
* **会话超时 `SESSION_EXPIRED`**：
  * 连接丢失 `CONNECTION_LOSS`后，在`session-timeout`指定时间内没有连接成功，则会话超时 `SESSION_EXPIRED`；
  * 会话超时，服务端认为会话结束，便开始清理会话，因为会话占用资源。但客户端并不知会话在服务端超时被清理，因此一直再重连，若真连上，服务端则直接发个状态会话超时 `SESSION_EXPIRED`。
* **会话转移 `SESSION_MOVED`** ：
  * 连接丢失 `CONNECTION_LOSS`后，在`session-timeout`指定时间内连接成功，且连接机器更换了（比如原来连接1号服务器，因为网络抖动失连，现在连接了2号服务器），这便是会话转移 `SESSION_MOVED` ；
  * 一旦发生会话转移 `SESSION_MOVED` ，客户端连接的具体机器有变化，客户端本身会做一些调整，把一些信息进行修改。

## 4.1.3 ACL

​	**ACL** 全称为 `Access Control List`(访问控制列表),用于控制资源的访问权限,是 zk 数据安全的保障。zk 利用 ACL 策略控制 znode 节点的访问权限,如**节点数据读写**、**节点创建**、**节点删除**、**读取子节点列表**、**设置节点权限**等。

### （1）ACL与UGO的对比

​	说到权限控制,我们最为熟悉的是 **Unix/Linux** 文件系统中使用的 **UGO**(`User`、`Group`、 `Others`)权限控制机制。即针对一个目录/文件,对创建者(User)、创建者所在组(Group) 及其它用户(Others)分配不同的权限。<u>UGO 是一种**粗粒度的文件系统权限控制模式**,其只能对三类用户进行权限控制</u>。但若想让某一与创建者(User)无关的组或用户拥有某些权限, UGO 是无法实现的。 

​	**ACL** 则是一种**细粒度的权限管理方式**,可以针对任意用户与组进行细粒度的权限控制。 目前大多数 Unix 已经支持了 ACL,Linux 也从 2.6 版本开始支持了 ACL。 

​	不过需要注意的是：**Linux 系统中文件或子目录默认会继承其父目录的 ACL。而在 Zookeeper 中,znode 的 ACL 是没有继承关系的,每个 znode 的权限都是独立控制的**,只有客户端满足 znode 设置的权限要求时,才能完成相应的操作。 

### （2）zk的ACL维度

​	**Linux** 系统的 **ACL** 分为两个维度:**组 `group`** 与**权限 `permission`**。而 Zookeeper 的 ACL 分为三个维度:**授权策略 `scheme`**、**授权对象 `id`**、**用户权限 `permission`**。 

#### A、授权策略scheme

​	授权策略用于**确定权限验证过程中使用的检验策略**,在 zk 中最常用的有<u>四种策略</u>：

* `IP`：根据 IP 进行权限验证 （有点儿类似防火墙，对IP进行权限限制）
* `digest`：（摘要）根据用户名密码进行验证。密码可以是明文,也可以是密文。 
* `world`：任何用户（anyone）在不做任何验证的情况下可以对==指定某一个 znode== 进行**任何操作 [只针对节点本身数据进行操作，不能操作权限]**
* `super`：超级用户可以对 zk 上==所有 znode== 进行**任意操作 [可以对权限进行操作]**

#### B、授权对象id

​	授权对象指的是权限赋予的用户。不同的授权策略具有不同类型的授权对象。下面是各个授权模式对应的授权对象。

* `ip`：授权对象是 IP 地址。
* `digest`：授权对象是“用户名:密码”。
* `world`：只有一个,即 anyone。
* `Super`：与 digest 模式相同 

#### C、权限Permission

​	权限指的是通过验证的用户可以对 znode 执行的操作。共有五种权限,不过 zk 支持自定义权限。 

* **c**：`Create`～创建
* **d**：`Delete`～删除
* **r**：`Read`～读取节点的数据内容及子节点列表
* **w**：`Write`～更新数据内容
* **a**：`acl`～数据节点 ACL 管理权限 

## 4.1.4 Watcher机制

​	zk 提供了分布式数据的发布订阅功能,一个发布者能够让多个订阅者同时监听某一主题对象,当这个主题对象状态发生变化时,会通知所有订阅者,使它们能够做出相应的处理。**zk 通过 Watcher 机制实现了发布订阅模式**。

### （1）watcher工作原理

![](https://ws2.sinaimg.cn/large/006tKfTcly1g0kso0htm6j313w0ksn40.jpg)

1. 首先，client会生成一个`watcher`对象（存储在本地`WatcherManager`对象中）；
2. 然后，向服务端注册Watcher（实质是由`WatcherManager`对象负责完成注册的）；
3. 当某个事件发生，则触发Watcher；
4. Watcher事件触发，服务端则向客户端发送相应的事件通知
5. 客户端根据相应的通知找到相应的watcher对象；
6. 最后，由找到的相应的watcher对象执行相应的回调。

注意⚠️：客户端生成 `watcher` 对象后会将其存储到本地的 `WatcherManager` 对象中,而该对象会将其注册到服务端。

### （2）watcher事件

对于同一个事件类型,在不同的通知状态中代表的含义是不同的。

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0n3vn2r0gj311z0u0nbr.jpg)

⚠️注意：

* `NodeChildrenChanged`：仅仅是指**子节点列表数量**改变，与内容无关；
* `AuthFailed`：用错用户名密码验证不会触发该事件，只有语法错误时才会触发；

### （3）watcher特性

​	zk 的 **watcher** 机制具有以下三个较明显的特性。 

* **一次性**：一旦一个 **watcher** 被触发,zk 就会从服务端删除。客户端再需要对服务端进行监听,则需要再注册 watcher。 
  * 官网表示，为了减轻zk压力，带宽压力并没有设计成一次注册长期有效，由此引申几个问题：
    * 如果watcher的事件触发很频繁，那么反复的watcher触发、注册创建watcher对客户端、服务端性能也有一定影响的？
    * 注册过程中，客户端感知不到的那些事件（watcher还未注册成功，就有事件发生）该如何处理？
  * 个人认为，利用zk的watcher机制需要依据场景
* **串行性**：**watcher** 回调方法的执行是串行的（第一个watcher回调执行未结束，第二个watcher回调就过来了，那么要待第一个执行完第二个才会执行，即为串行）。 
* **轻量级**：客户端向服务端注册 **watcher**,其并没有将 **watcher** 发送到服务端(存放在了客户端的`watcherManager` 中),而是向服务端发送了一个 **watcher** 注册信号。另外, **watcher** 的回调逻辑存在于客户端，而没有在服务端。 

# 4.2 客户端命令

## 4.2.1 启动客户端

### （1）连接本机zk服务器

![](https://ws3.sinaimg.cn/large/006tKfTcly1g0ksx0l708j30ns06wjsq.jpg)

### （2）连接其他zk服务器

![](https://ws1.sinaimg.cn/large/006tKfTcly1g0ksxftx3jj313a06yacb.jpg)

## 4.2.2 查看子节点`-ls`

### （1）命令语法

* 语法：`ls path`
* 说明：查看指定节点所包含的所有子节点。

### （2）查看操作

查看根节点及/brokers 节点下所包含的所有子节点列表。

![](https://ws1.sinaimg.cn/large/006tKfTcly1g0ksymaoglj312y0em7ci.jpg)

## 4.2.3 创建节点`-create`

### （1）命令语法

语法:`create [-s] [-e] path data acl`

* `-s`：**创建有序节点**。如果在创建 znode 时,我们使用排序标志的话,ZooKeeper 会在我们指定的 znode 名字后面增加一个数字。我们继续加入相同名字的 znode 时,这个数字会不断增加。这个序号的计数器是由这些排序znode 的父节点来维护的。
* `-e`：**创建临时节点**。节点类型一旦确定就不能修改了。
* `path`：指定要创建的 znode 名称及其路径。注意：该路径中的所有 znode 必须是已经存在的。
* `acl`：权限控制,默认情况下不做任何权限控制。 

### （2）创建永久节点

关于 ACL 权限控制,后面会专门讲解。 

创建一个名称为 `china` 的 **znode**,其值为 `999`。 

![](https://ws3.sinaimg.cn/large/006tKfTcly1g0kt7n7m4dj30o40b4jug.jpg)

### （3）创建顺序节点

在`/china` 节点下创建了顺序子节点 `beijing`、`shanghai`、`guangzhou`,它们的数据内容分别为 `bj`、`sh`、`gz`。 

![](https://ws4.sinaimg.cn/large/006tKfTcly1g0kt8j91pkj313i0cs7dd.jpg)

### （4）创建临时节点

临时节点与持久节点的区别：在后面 `get` 命令中可以看到。

![](/Users/changbingbing/Library/Application Support/typora-user-images/image-20190227111613957.png)

## 4.2.4 获取节点数据内容`-get`

### （1）命令语法

* 语法:`get path` 
* 说明:获取指定节点的数据内容及属性信息。 

### （2）获取持久节点数据

![](https://ws1.sinaimg.cn/large/006tKfTcly1g0kteet85fj312m0n0k0s.jpg)

ephemeralOwner 为 0,说明其为持久节点。数据值为 999,数据长度为 3,子节点数为 6。

### （3）获取顺序节点信息

![](https://ws1.sinaimg.cn/large/006tKfTcly1g0ktf2s4goj313a0h0q9m.jpg)

从节点属性信息中看不出是否是顺序节点,但从节点名称中可以看出。

### （4）获取临时节点信息

![](https://ws4.sinaimg.cn/large/006tKfTcly1g0ktfullv2j30y20u04dx.jpg)

​	这两个节点的 `ephemeralOwner` 值都不为 0,且值相同,为什么会相同呢?其值为创建该节点的会话的`SessionID`。当会话消失后,会根据 `SessionID` 来查找与该会话相关的临时节点进行删除。 

​	通过 `quit` 命令退出客户端后,再次连接上 **zookeeper**,然后再查看`/china` 的节点列表, 会发现原来的节点`/china/aaa`、`/china/bbb` 与`/china/ccc` 均消失。这就是临时节点,会随着会话的关闭而消失。 

## 4.2.5 更新节点数据内容`-set`

### （1）命令语法

* 语法:`set path data` 
* 说明:修改指定节点的数据值 

### （2）更新数据

* 更新前：
  ![](https://ws2.sinaimg.cn/large/006tKfTcly1g0ktjhb7r6j312i0na7e0.jpg)
* 更新：
  ![](https://ws1.sinaimg.cn/large/006tKfTcly1g0ktju5lotj312q0kqdoa.jpg)
* 更新后发现 dataVersion 的值由 0 变为了 1。再查看该节点的值。
  ![](https://ws2.sinaimg.cn/large/006tKfTcly1g0ktk7wb06j312u0n6n6a.jpg)

## 4.2.6 删除节点`-delete`

### （1）命令语法

* 语法:`delete path`
  说明：删除指定节点。不过需要注意,只能删除其下没有子节点的节点。

### （2）删除操作

![](https://ws4.sinaimg.cn/large/006tKfTcly1g0ktmvyrsgj312u08q431.jpg)

若要删除具有子节点的节点,会报错。

![](https://ws1.sinaimg.cn/large/006tKfTcly1g0ktnebld9j3126084tc9.jpg)

## 4.2.7 ACL操作

### （1）查看权限`-getAcl`

![](https://ws4.sinaimg.cn/large/006tKfTcly1g0ktnw0784j312m09kgoz.jpg)

注意：命令是大小写敏感的。`/china` 节点的授权策略是 `world`,授权对象是 `anyone`,权
限是 `cdrwa` 所有权限。

### （2）设置权限

这里使用密码为明文的权限设置方式。其需要两步:

* 首先增加一个认证用户,使用命令为: `addauth digest 用户名:密码明文`
* 然后再设置权限,使用命令为：`setAcl /path auth:用户名:密码明文:权限` 

​	下面的命令是,首先增加了一个认证用户 zs,密码为 123,然后为`/china` 节点指定只有 zs 用户才可访问该节点,而访问权限为所有权限。且设置过后,其 **aclVersion** 增加了 1。

![](https://ws4.sinaimg.cn/large/006tKfTcly1g0ktti4qudj312g0gk463.jpg)

​	不使用授权用户来查看节点,其会显示如下内容,表示 zs 用户具有 cdrwa 权限。但密码显示的内容为 123 加密过的内容。

![](https://ws1.sinaimg.cn/large/006tKfTcly1g0kttqmidvj31200a4td1.jpg)

​	通过 `quit` 退出客户端后,再次进入客户端,然后获取`/china` 的数据时发现,其给出的提示是权限无效。此时需要再次在客户端中通过 `addauth` 添加 zs 用户,然后才可以访问。

![](https://ws1.sinaimg.cn/large/006tKfTcly1g0ktu29czij312m0o2alm.jpg)

### （3）验证权限不继承

​	通过 `quit` 退出客户端后,再次进入客户端,然后获取`/china` 的数据时其肯定会提示权限无效。但查看其子节点`/china/beijing0000000001` 时发现是可以查看的,说明子节点没有继承父节点的 ACL 权限。

![](https://ws1.sinaimg.cn/large/006tKfTcly1g0ktvdrwjpj312m0js10v.jpg)

4.3 ZKClient客户端

4.4 Curator客户端