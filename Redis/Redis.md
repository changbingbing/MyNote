# Redis介绍

## 什么是Redis？

* **`Redis`**是用**==C语言==**开发的一个开源的**高性能**、**==键值对==**（`key-value`）**==内存数据库==**。
* 它提供了**==五种数据类型==**来存储键值对中的**值**：<u>字符串类型、散列类型、列表类型、集合类型、有序集合类型</u>
* 它是一种**`NoSQL`**数据库。

## 什么是NoSQL？

* `NoSQL`，即`Not-Only SQL`（不仅仅是`SQL`），泛指**非关系型的数据库**。

* 什么是**关系型**数据库？**数据结构是一种有行有列的数据库**

* `NoSQL`数据库是为了解决**高并发、高可用、高可扩展、大数据存储**问题而产生的数据库解决方案。

* `NoSQL`可以作为关系型数据库的良好补充，但是**不能替代关系型数据库**。

## NoSQL数据库分类

* **键值(Key-Value)存储数据库**

相关产品： `Tokyo Cabinet/Tyrant`、**`Redis`**、`Voldemort`、`Berkeley DB`

典型应用： 内容缓存，主要用于处理大量数据的高访问负载。

数据模型： 一系列键值对

优势： **快速查询**

劣势： 存储的数据缺少结构化

* **列存储数据库**

相关产品：`Cassandra`, **`HBase`**, `Riak`

典型应用：分布式的文件系统

数据模型：以列簇式存储，将同一列数据存在一起

优势：查找速度快，可扩展性强，更容易进行分布式扩展

劣势：功能相对局限

* **文档型数据库**

相关产品：`CouchDB`、**`MongoDB`**

典型应用：`Web`应用（与`Key-Value`类似，`Value`是结构化的）

数据模型： 一系列键值对

优势：数据结构要求不严格(数据操作更加灵活，文档就类似于一个java对象，有属性和属性值，而属性值又可以是一个对象)

劣势：

* 图形(`Graph`)数据库

相关数据库：`Neo4J`、`InfoGrid`、`Infinite Graph`

典型应用：社交网络

数据模型：图结构

优势：利用图结构相关算法。

劣势：需要对整个图做计算才能得出结果，不容易做分布式的集群方案。

## Redis发展历史

**2008**年，意大利的一家创业公司`Merzia`推出了一款基于`MySQL`的网站实时统计系统`LLOOGG`，然而没过多久该公司的创始人 **`Salvatore Sanfilippo`**便 对`MySQL`的性能感到失望，于是他决定亲自为`LLOOGG`量身**定做一个数据库，并于2009年开发完成，这个数据库就是`Redis`**。 

不过`Salvatore Sanfilippo`并不满足只将`Redis`用于`LLOOGG`这一款产品，而是希望更多的人使用它，于是在**同一年`Salvatore Sanfilippo`将`Redis`开源发布**，并开始和`Redis`的另一名主要的代码贡献者`Pieter Noordhuis`一起**继续着`Redis`的开发，直到今天**。

`Salvatore Sanfilippo`自己也没有想到，短短的几年时间，`Redis`就拥有了庞大的用户群体。`Hacker News`在2012年发布了一份数据库的使用情况调查，结果显示有近12%的公司在使用Redis。**国内如新浪微博、街旁网、知乎网，国外如`GitHub`、`Stack Overflow`、`Flickr`等都是`Redis`的用户。**

`VMware`公司从**2010年**开始赞助`Redis`的开发， `Salvatore Sanfilippo`和`Pieter Noordhuis`也分别**在3月和5月加入`VMware`，全职开发`Redis`**。

## Redis应用场景

- **内存数据库**（登录信息、购物车信息、用户浏览记录等）
- **缓存服务器**（商品数据、广告数据等等）（**最多使用**）
- 解决分布式集群架构中的`session`分离问题（`session`共享）
- 任务队列（秒杀、抢购、12306等等）
- 支持发布订阅的消息模式
- 应用排行榜
- 网站访问统计
- 数据过期处理（可以精确到毫秒）

# Redis单机版安装配置

## Redis下载

* 官网地址：<http://redis.io/>

* 中文官网地址：<http://www.redis.cn/>

* 下载地址：[http://download.redis.io/releases/](http://download.redis.io/releases/redis-3.0.0.tar.gz) 

![1545818367657](https://ws1.sinaimg.cn/large/006tNbRwgy1fynfcbbdgoj309004qjrf.jpg)                                                           

![1545818382242](https://ws2.sinaimg.cn/large/006tNbRwgy1fynfcn1327j30fg02qmxn.jpg)       

## Redis安装环境

`Redis`没有官方的`Windows`版本，所以建议在`Linux`系统（CentOS、Ubuntu、RedHat等）上安装运行，我们使用`CentOS7-64位`（[`Linux`](https://www.cnblogs.com/cb0327/p/5396200.html)操作系统的一个系列）作为安装环境。

![1545818400322](https://ws3.sinaimg.cn/large/006tNbRwgy1fynfdgznzdj30fg01vdg2.jpg)

* 购买云服务器（阿里云、腾讯云等）
* 使用VMware在自己电脑中玩虚拟机
* docker运行在Ubuntu操作系统或者CentOS操作系统。

## Redis安装

第一步：安装`C`语言需要的`GCC`环境

```bash
yum  install -y gcc-c++   
yum install -y wget
```

第二步：下载并解压缩`Redis`源码压缩包

```bash
wget http://download.redis.io/releases/redis-3.2.9.tar.gz
tar -zxf redis-3.2.9.tar.gz   
```

第三步：编译`Redis`源码，进入`redis-3.2.9`目录，执行编译命令

```bash
cd redis-3.2.9
make   
```

第四步：安装`Redis`，通过`PREFIX`指定安装路径

```bash
make   install PREFIX=/kkb/server/redis   
```

> [Mac Redis安装、启动](https://www.cnblogs.com/cb0327/p/7326963.html)

## Redis启动

### 前端启动

* 启动命令：**`redis-server`**，直接运行`bin/redis-server`将以前端模式启动

  ```shell
  ./redis-server
  ```

* 关闭命令：`ctrl+c`

* 启动缺点：客户端窗口关闭则`redis-server`程序结束，**不推荐使用此方法**

* 启动图例：

![1545818750862](https://ws3.sinaimg.cn/large/006tNbRwgy1fynfsndsrnj30ff08jad0.jpg)          

### 后端启动（守护进程启动）－推荐

* 第一步：拷贝`redis-3.2.9/redis.conf`配置文件到`Redis`安装目录的`bin`目录

  ```bash
  cp  redis.conf /kkb/server/redis/bin/ 
  ```

* 第二步：修改`redis.conf`

  ```bash
  vim redis.conf
  ```

  ```properties
   # 将`daemonize`由`no`改为`yes`，是否用守护线程方式启动
    daemonize yes
    
    # 默认绑定的是回环地址，默认不能被其他机器访问，因此若要设置主从、集群，这里要绑定实际地址方能被其他机器访问
    # bind 127.0.0.1
    
    # 是否开启保护模式，由yes该为no
    protected-mode no 
  ```


* 第三步：启动服务

  ```bash
  ./redis-server redis.conf
  ```

* 后端启动的关闭方式

  ```bash
  ./redis-cli shutdown
  ```

## 其他命令说明

- `redis-server` ：启动`redis`服务
- `redis-cli` ：进入`redis`命令客户端
- `redis-benchmark`： 性能测试的工具
- `redis-check-aof` ： `aof`文件进行检查的工具
- `redis-check-dump` ：  `rdb`文件进行检查的工具
- `redis-sentinel` ：  启动哨兵监控服务

# Redis命令行客户端

![1545819295016](https://ws1.sinaimg.cn/large/006tNbRwgy1fynlrvpj0gj30fd02z3zv.jpg)                                                  

* **命令格式**

```bash
./redis-cli -h 127.0.0.1 -p 6379
```

* **参数说明**

```
-h：redis服务器的ip地址
-p：redis实例的端口号
```

* **默认方式**

  **如果不指定主机和端口也可以**

  * 默认主机地址是127.0.0.1 
  * 默认端口是6379

  ```bash
  ./redis-cli   
  ```


# Redis数据类型

官方命令大全网址：<http://www.redis.cn/commands.html>

## 数据类型介绍

`Redis`中存储数据是通过`key-value`格式存储数据的，其中`value`可以定义五种数据类型：

* **String（字符类型）**

* **Hash（散列类型）**

* **List（列表类型）**

* **Set（集合类型）**

* **SortedSet（有序集合类型，简称zset）**

注意：在`redis`中的命令语句中：

1. **命令是忽略大小写的，而`key`是不忽略大小写的。**
2. key是唯一的
   1. 若不唯一且value类型相同，不会报错，但会直接覆盖；
   2. 若不唯一且value类型不同，会报错
   3. 

![数据类型理解](https://ws3.sinaimg.cn/large/006tNbRwgy1fys9qnl3g2j30ye0elglz.jpg)



## string类型

### 命令

> set、setnx、get、getset、incr、incrby、decr、decrby、append、strlen、mset、mget

#### 赋值：SET

* 语法：

```
SET key value
```

* 示例：

```bash
127.0.0.1:6379> set test 123 
OK   
```

#### 赋值：SETNX

**==仅当不存在时进行赋值==，使用该命令可以实现分布式锁的功能，后续讲解！！！**

* 语法：

```bash
setnx key value
```

* 示例：

```bash
redis> EXISTS job                # job 不存在
(integer) 0
redis> SETNX job "programmer"    # job 设置成功
(integer) 1
redis> SETNX job "code-farmer"   # 尝试覆盖 job ，失败
(integer) 0
redis> GET job                   # 没有被覆盖
"programmer"
```

#### 取值：GET

* 语法：

```
GET key
```

* 示例：

```bash
127.0.0.1:6379> get test
"123"
```

#### 取值并赋值：GETSET

* 语法：


```
GETSET key value
```

* 示例：


```bash
127.0.0.1:6379> GETSET test 456
"123"
127.0.0.1:6379> get test
"456"
```

#### 数值增减

**注意**

* incr、incrby、decr、decrby：只能针对**整数类型**的数据进行使用。 

* incr命令是**【原子】操作**。可用来生成数据库表的自增主键，是非常**安全且高效**。
* **使用场景**：分布式数据库中订单ID的生成。

```java
//以下代码不是一个原子性操作，就可能存在线程安全问题。	
int i = 1;
i++;
System.out.println(i)
```

##### 递增数值：INCR 

* 语法：

```
INCR key
```

* 示例：

```bash
127.0.0.1:6379> incr num
(integer) 1
127.0.0.1:6379> incr num
(integer) 2
127.0.0.1:6379> incr num
(integer) 3
```

##### 按指定步长递增：INCRBY

* 语法：

```
INCRBY key increment
```

* 示例：

```bash
127.0.0.1:6379> incrby num 2
(integer) 5
127.0.0.1:6379> incrby num 2
(integer) 7
127.0.0.1:6379> incrby num 2
(integer) 9
```

##### 递减数值：DECR

* 语法：

```
DECR key
```

* 示例：

```bash
127.0.0.1:6379> get num
"9"
127.0.0.1:6379> DECR num
(integer) 8
127.0.0.1:6379> DECR num
(integer) 7
```

##### 按指定步长递减： DECRBY

* 语法：

```
DECRBY key decrement
```

* 示例

```bash
127.0.0.1:6379> DECRBY num 3
(integer) 4
127.0.0.1:6379> DECRBY num 3
(integer) 1
127.0.0.1:6379> get num
"1"
```

#### 其它命令

##### 尾部追加值 ：APPEND

`APPEND`命令：键存在，则向键值末尾追加`value`；不存在，则将该键的值设置为`value`（效果相当于 `SET key value`）。其返回值是`value`字符串的总长度。 

* 语法：

```bash
APPEND key value
```

* 示例：

```bash
127.0.0.1:6379> set str hello
OK
127.0.0.1:6379> append str " world!"
(integer) 12
127.0.0.1:6379> get str 
"hello world!"
```

##### 获取字符串长度 ：STRLEN

`STRLEN`命令，返回键值的长度，如果键不存在则返回0。

* 语法：

```
STRLEN key
```

* 示例：

```bash
127.0.0.1:6379> strlen str 
(integer) 0
127.0.0.1:6379> set str hello
OK
127.0.0.1:6379> strlen str 
(integer) 5
```

##### 同时设置/获取多个键值 ：MSET/MGET

* 语法：

```base
MSET key value [key value …]
MGET key [key …]
```

* 示例：

```bash
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3
OK
127.0.0.1:6379> get k1
"v1"
127.0.0.1:6379> mget k1 k3
1) "v1"
2) "v3"
```

### 应用场景之自增主键

* **需求：商品编号、订单号采用`INCR`命令生成。**

* **设计：**`key`命名要有一定的设计

* **实现：**定义商品编号`key`：`items:id`

```bash
192.168.101.3:7003> INCR items:id
(integer) 2
192.168.101.3:7003> INCR items:id
(integer) 3
```

## hash类型

### hash类型介绍

`hash`类型也叫**散列类型**，它提供了**字段**和**字段值**的映射。字段值只能是**字符串**类型，不支持散列类型、集合类型等其它类型。如下：

![1545823253166](https://ws4.sinaimg.cn/large/006tNbRwgy1fynlsv1nchj30dx06g3yi.jpg)             

* `hash`类型和`string`类型区别
  * `hash`类型适合于**增删改**操作。
  * `string`类型适合于**查询**操作。string类型存储对象，需要进行**对象转换为json串**进行存储。​                              

### 命令

> **hset、hget、hdel、hmget、hmset**、hincrby、hexists、hkeys、hvals、hlen

#### 赋值

`HSET`命令不区分插入和更新操作，当执行插入操作时`HSET`命令返回`1`，当执行更新操作时返回`0`。 

##### 设置一个字段值：HSET

* 语法：

```
HSET key field value
```

* 示例：

```bash
127.0.0.1:6379> hset user username zhangsan 
(integer) 1
```

##### 设置多个字段值：HMSET

* 语法：

```
HMSET key field value [field value ...]
```

* 示例：

```bash
127.0.0.1:6379> hmset user age 20 username lisi 
OK
```

##### 赋值：HSETNX

类似`HSET`，区别在于当字段不存在时赋值；如果字段存在，该命令不执行任何操作

* 语法：

```
HSETNX key field value
```

* 示例：

```bash
127.0.0.1:6379> hsetnx user age 30	# 如果user中没有age字段则设置age值为30，否则不做任何操作
(integer) 0

```

#### 取值 

##### 获取一个字段值：HGET

* 语法：

```
HGET key field
```

* 示例：

```bash
127.0.0.1:6379> hget user username   
"zhangsan"  
```

##### 获取多个字段值：HMGET

* 语法：        

```
HMGET key field [field ...]
```

* 示例：

```bash
127.0.0.1:6379> hmget user age username   
1) "20"   
2) "lisi"   
```

##### 获取所有字段值：HGETALL

* 语法：

```
HGETALL key
```

* 示例：

```bash
127.0.0.1:6379> hgetall user
1) "age"
2) "20"
3) "username"
4) "lisi"
```

#### 删除字段：HDEL

可以删除一个或多个字段，返回值是被删除的字段个数 

* 语法：

```
HDEL key field [field ...]
```

* 示例：

```bash
127.0.0.1:6379> hdel user age
(integer) 1
127.0.0.1:6379> hdel user age name
(integer) 0
127.0.0.1:6379> hdel user age username
(integer) 1 
```

#### 数值递增 ：HINCRBY

* 语法：

```
HINCRBY key field increment
```

* 示例：

```bash
127.0.0.1:6379> hincrby user age 2	# 将用户的年龄加2
(integer) 22
127.0.0.1:6379> hget user age		# 获取用户的年龄
"22"
```

#### 其它命令(自学)

##### 判断字段是否存在：HEXISTS

* 语法：

```
HEXISTS key field
```

* 示例：

```bash
127.0.0.1:6379> hexists user age	#查看user中是否有age字段
(integer) 1
127.0.0.1:6379> hexists user name	#查看user中是否有name字段
(integer) 0
```

##### 只获取字段名或字段值：HKEYS/HVALS

* 语法：

```
HKEYS key
HVALS key
```

* 示例：

```
127.0.0.1:6379> hmset user age 20 name lisi 
OK
127.0.0.1:6379> hkeys user
1) "age"
2) "name"
127.0.0.1:6379> hvals user
1) "20"
2) "lisi"
```

##### 获取字段数量 ：HLEN

* 语法：

```
HLEN key
```

* 示例：

```bash
127.0.0.1:6379> hlen user   
(integer) 2
```

### 应用之存储商品信息

> 注意事项：存储那些对象数据，特别是对象属性经常发生增删改操作的数据。

* 商品信息字段

  ```
  【商品id、商品名称、商品描述、商品库存、商品好评】
  ```


* 定义商品信息的key

  ```
  商品ID为1001的信息在 Redis中的key为：[items:1001]
  ```


* 存储商品信息

  ```bash
  192.168.101.3:7003> HMSET items:1001 id 3 name apple price 999.9
  OK
  ```


* 获取商品信息   

  ```bash
  192.168.101.3:7003> HGET items:1001 id
  "3"
  192.168.101.3:7003> HGETALL items:1001
  1) "id"
  2) "3"
  3) "name"
  4) "apple"
  5) "price"
  6) "999.9"
  ```


## list类型

### ArrayList与LinkedList的区别

* `ArrayList`使用**数组方式**存储数据，所以根据索引**查询数据速度快**，而**新增或者删除**元素时需要涉及到位移操作，所以**比较慢**。 （即**==查询快、增删慢==**）

* `LinkedList`使用**双向链表方式**存储数据，每个元素都记录前后元素的指针，所以**插入、删除数据**时只是更改前后元素的指针指向即可，**速度非常快**。然后通过下标**查询元素**时需要从头开始索引，所以**比较慢**，但是如果查询前几个元素或后几个元素速度比较快。（即**==查询慢、增删快==**）

![1545827176006](https://ws3.sinaimg.cn/large/006tNbRwgy1fynmrhai6cj30fj094gmq.jpg) 

![1545827181696](https://ws1.sinaimg.cn/large/006tNbRwgy1fynmrolcisj30fh08075g.jpg) 

### list类型介绍

* `Redis`的列表类型（`list`类型）可以`存储一个有序的字符串列表`，常用的操作是向列表两端添加元素，或者获得列表的某一个片段。

* 列表类型内部是使用**双向链表（`double linked list`）**实现的，所以向列表两端添加元素的时间复杂度为`0(1)`，获取越接近两端的元素速度就越快。这意味着即使是一个有几千万个元素的列表，获取头部或尾部的10条记录也是极快的。 

  > * 适合于**只对list列表两端进行操作**的场景。
  > * list类型存储的数据特点：**==有序可重复==**（指的是**==插入顺序，而不是自然排序顺序==**）
  > * 可以用来作为**消息队列**去使用
  > * 可以用来**实现商品评论表**

### 命令

> **lpush、lpop、rpush、rpop、lrange**、llen、

#### LPUSH/RPUSH

* 语法：

```
LPUSH key value [value ...]
RPUSH key value [value ...]
```

* 示例：

```bash
127.0.0.1:6379> lpush list:1 1 2 3   
(integer) 3   
127.0.0.1:6379> rpush list:1 4 5 6   
(integer) 3   
```

#### LRANGE

> **获取列表中的某一片段**。将返回`start`、`stop`之间的所有元素（**包含两端的元素**），索引从`0`开始。索引可以是负数，如：“`-1`”代表最后边的一个元素。

* 语法：

```
LRANGE key start stop
```

* 示例：

```bash
127.0.0.1:6379> lrange   list:1 0 2   
1) "2"   
2) "1"   
3) "4"   
```

#### LPOP/RPOP

> 从列表两端弹出元素

从列表左边弹出一个元素，会分两步完成：

* 第一步是将列表左边的元素从列表中移除

* 第二步是返回被移除的元素值。

语法：

```
LPOP key
RPOP key
```

示例

```bash
127.0.0.1:6379>lpop list:1   
"3"  
127.0.0.1:6379>rpop list:1   
"6"  
```

#### LLEN

> 获取列表中元素的个数

* 语法：

```
LLEN key
```

* 示例：

```bash
127.0.0.1:6379> llen list:1   
(integer) 2   
```

#### 其它命令(自学)

##### LREM

> 删除列表中指定个数的值 

​	`LREM`命令会删除列表中前`count`个值为`value`的元素，返回实际删除的元素个数。根据`count`值的不同，该命令的执行方式会有所不同： 

```
 -  当count>0时， LREM会从列表左边开始删除。 
 -  当count<0时， LREM会从列表后边开始删除。 
 -  当count=0时， LREM删除所有值为value的元素。 
```

* 语法：

```
LREM key count value
```

* 示例 

```bash
127.0.0.1:6379> LRANGE list 0 -1
1) "1"
2) "1"
3) "3"
4) "5"
5) "5"
6) "3"
7) "1"
8) "8"
9) "5"
127.0.0.1:6379> LREM list -2 5
(integer) 2
127.0.0.1:6379> LRANGE list 0 -1
1) "1"
2) "1"
3) "3"
4) "5"
5) "3"
6) "1"
7) "8"
```

##### LINDEX

>  获得指定索引(起始值为0)的元素值

* 语法：

```
LINDEX key index
```

* 示例：

```bash
127.0.0.1:6379>lindex l:list 2   
"1"   
```

##### LSET

> 设置指定索引的元素值

* 语法：

  ```bash
  LSET key index value
  ```

* 示例：

  ```bash
  127.0.0.1:6379>lset l:list 2 2
  OK   
  127.0.0.1:6379>lrange l:list 0 -1   
  1)"6"   
  2)"5"   
  3)"2"   
  4)"2"   
  ```

##### LTRIM

> 只保留列表指定片段,指定范围和LRANGE一致 

* 语法：

```
LTRIM key start stop
```

* 示例：

```bash
127.0.0.1:6379> lrange l:list 0 -1
1) "6"
2) "5"
3) "0"
4) "2"
127.0.0.1:6379> ltrim l:list 0 2
OK
127.0.0.1:6379> lrange l:list 0 -1
1) "6"
2) "5"
3) "0"
```

##### LINSERT 

> 向列表中指定位置插入元素。
> 该命令首先会在列表中从左到右查找值为pivot的元素，然后根据第二个参数是BEFORE还是AFTER来决定将value插入到该元素的前面还是后面。

语法：

```
LINSERT key BEFORE|AFTER pivot value
```

示例：

```bash
127.0.0.1:6379> lrange list 0 -1
1) "3"
2) "2"
3) "1"
127.0.0.1:6379> linsert list after 3 4
(integer) 4
127.0.0.1:6379> lrange list 0 -1
1) "3"
2) "4"
3) "2"
4) "1"
```

##### RPOPLPUSH

> 从源列表右边弹出一个元素推入新列表左侧

* 语法：

```
RPOPLPUSH source destination
```

* 示例：

```
127.0.0.1:6379> rpoplpush list newlist 
"1"
127.0.0.1:6379> lrange newlist 0 -1
1) "1"
127.0.0.1:6379> lrange list 0 -1
1) "3"
2) "4"
3) "2"
```

### 应用之商品评论列表

* 需求：

```
用户针对某一商品发布评论，一个商品会被不同的用户进行评论，存储商品评论时，要按时间顺序排序。
用户在前端页面查询该商品的评论，需要按照时间顺序降序排序。
```

* 分析：

```
使用list存储商品评论信息，KEY是该商品的ID，VALUE是商品评论信息列表
```

* 实现：商品编号为`1001`的商品评论`key`【`items: comment:1001`】

```bash
192.168.101.3:7001> LPUSH items:comment:1001 '{"id":1,"name":"商品不错，很好！！","date":1430295077289}'   
```

## set类型

### set类型介绍

* `set`类型即**集合类型**，其中的数据是**==无序不重复==**。
* 集合类型和列表类型的对比：
  ![1545978348626](https://ws1.sinaimg.cn/large/006tNbRwgy1fynoslg9yjj30fe02sjrs.jpg)   

* 集合类型的常用操作是向集合中加入或删除元素、判断某个元素是否存在等，由于集合类型的`Redis`内部是使用值为空的散列表实现，所有这些操作的时间复杂度都为`0(1)`。 


* `Redis`还提供了多个集合之间的**交集、并集、差集**的运算。

### 基础命令

> sadd 、srem、smembers、sismember

#### SADD/SREM

> 添加元素/删除元素

* 语法：


```
SADD key member [member ...]
SREM key member [member ...]
```

* 示例：


```bash
127.0.0.1:6379> sadd set a b c   
(integer) 3   
127.0.0.1:6379> sadd set a   
(integer) 0   
127.0.0.1:6379> srem set c d   
(integer) 1   
```

#### SMEMBERS

> 获得集合中的所有元素

* 语法：


```
SMEMBERS key
```

* 示例：


```bash
127.0.0.1:6379> smembers set   
1)   "b"   
2)   "a"
```

#### SISMEMBER

> 判断元素是否在集合中

* 语法：


```
SISMEMBER key member
```

* 示例：


```bash
127.0.0.1:6379>sismember set a   
(integer)   1   
127.0.0.1:6379>sismember set h   
(integer)   0   
```

### 集合运算命令

> sdiff、sinter、sunion

#### SDIFF

> 集合的差集运算 A-B：属于A并且不属于B的元素构成的集合。 

![1545980186479](https://ws2.sinaimg.cn/large/006tNbRwgy1fynpc76uk0j304b04taad.jpg)   

* 语法：


```
SDIFF key [key ...]
```

* 示例：


```bash
127.0.0.1:6379> sadd setA 1 2 3
(integer) 3
127.0.0.1:6379> sadd setB 2 3 4
(integer) 3
127.0.0.1:6379> sdiff setA setB 
1) "1"
127.0.0.1:6379> sdiff setB setA 
1) "4"
```

#### SINTER

> 集合的交集运算 A ∩ B：属于A且属于B的元素构成的集合。 

![1545980214245](https://ws4.sinaimg.cn/large/006tNbRwgy1fynpchx29qj304h04uwet.jpg)   

* 语法：


```
SINTER key [key ...]
```

* 示例：


```bash
127.0.0.1:6379> sinter setA setB 
1) "2"
2) "3"
```

#### SUNION

> 集合的并集运算 A ∪ B：属于A或者属于B的元素构成的集合

* 语法：


```
SUNION key [key ...]
```

* 示例：


```bash
127.0.0.1:6379> sunion setA setB
1) "1"
2) "2"
3) "3"
4) "4"
```

### 其它命令(自学)

> scard、spop

#### SCARD 

> 获得集合中元素的个数 

* 语法：


```
SCARD key
```

* 示例：


```bash
127.0.0.1:6379> smembers setA 
1) "1"
2) "2"
3) "3"
127.0.0.1:6379> scard setA 
(integer) 3
```

#### SPOP

> 从集合中弹出一个元素。
> 注意：由于集合是无序的，所有SPOP命令会从集合中随机选择一个元素弹出

* 语法：


```
SPOP key
```

* 示例：


```bash
127.0.0.1:6379> spop setA 
"1"
```

## zset类型 （sortedset）

### zset介绍

* 在`set`集合类型的基础上，**有序集合**类型为集合中的每个元素都`关联一个分数`，这使得我们不仅可以完成插入、删除和判断元素是否存在在集合中，还能够获得分数最高或最低的前N个元素、获取指定分数范围内的元素等与分数有关的操作。 
  * zset类型存储的数据特点：**不重复、有序**。
  * 底层还是一个set集合，不过，该集合中给**每个member设置一个score，通过score进行排序**
* 有序集合和列表对比
  * 相似处：
    * 有序
    * 可以获取指定范围元素
  * 区别处：
    * 列表类型是通过**链表**实现的，获取靠近两端的数据速度极快，而当元素增多后，访问中间数据的速度会变慢。
    * 有序集合类型使用**散列表**实现，所有即使读取位于中间部分的数据也很快。
    * 列表中不能简单的调整某个元素的位置，但是有序集合可以（**通过更改分数实现**）
    * **==有序集合要比列表类型更耗内存==**。
* 使用场景：销售排行榜
  * 销量作为分数
  * 销售人员或者商品作为member

### 命令

> zadd、zrem、zrange、zscore

#### ZADD

> 增加元素。
>
> 向有序集合中加入一个元素和该元素的分数，如果该元素已经存在则会用新的分数替换原有的分数。返回值是新加入到集合中的元素个数，不包含之前已经存在的元素。 

* 语法：


```
ZADD key score member [score member ...]
```

* 示例：


```bash
127.0.0.1:6379> zadd scoreboard 80 zhangsan 89 lisi 94 wangwu 
(integer) 3
127.0.0.1:6379> zadd scoreboard 97 lisi 
(integer) 0
```

#### ZREM 

> 删除元素。
>
> 移除有序集合key中的一个或多个成员，不存在的成员将被忽略。
> 当key存在但不是有序集类型时，返回一个错误。

* 语法：

```
ZREM key member [member ...]

```

* 示例：

```bash
127.0.0.1:6379> zrem scoreboard lisi
(integer) 1
```

#### ZRANGE/ZREVRANGE

> 获得排名在某个范围的元素列表。
>  - ZRANGE：按照元素分数从小到大的顺序返回索引从start到stop之间的所有元素（包含两端的元素）
>  - ZREVRANGE：按照元素分数从大到小的顺序返回索引从start到stop之间的所有元素（包含两端的元素）

* 语法：


```
ZRANGE key start stop [WITHSCORES]  
ZREVRANGE key start stop [WITHSCORES]     
```

* 示例： 


```bash
127.0.0.1:6379> zrange scoreboard 0 2
1) "zhangsan"
2) "wangwu"
3) "lisi"
127.0.0.1:6379> zrevrange scoreboard 0 2
1) "lisi "
2) "wangwu"
3) " zhangsan" 
```

* 如果需要获得元素的分数的可以在命令尾部加上`WITHSCORES`参数 

```
127.0.0.1:6379> zrange scoreboard 0 1 WITHSCORES
1) "zhangsan"
2) "80"
3) "wangwu"
4) "94"
```

#### ZSCORE

> 获取元素的分数。

* 语法：


```
ZSCORE key member
```

* 示例：


```bash
127.0.0.1:6379> zscore scoreboard lisi 
"97"
```

#### 其它命令(自学)

##### ZRANGEBYSCORE

> 获得指定分数范围的元素。

* 语法：


```
ZRANGEBYSCORE key min max [WITHSCORES]
```

* 示例：


```bash
127.0.0.1:6379> ZRANGEBYSCORE scoreboard 90 97 WITHSCORES
1) "wangwu"
2) "94"
3) "lisi"
4) "97"
127.0.0.1:6379> ZRANGEBYSCORE scoreboard 70 100 limit 1 2
1) "wangwu"
2) "lisi"
```

##### ZINCRBY 

> 增加某个元素的分数。
> 返回值是更改后的分数

* 语法：


```
ZINCRBY  key increment member
```

* 示例：


```
127.0.0.1:6379> ZINCRBY scoreboard 4 lisi 
"101"
```

##### ZCARD 

> 获得集合中元素的数量。

语法：

```
ZCARD key
```

示例：

```bash
127.0.0.1:6379> ZCARD scoreboard   
(integer)   3  
```

##### ZCOUNT

> 获得指定分数范围内的元素个数 

* 语法：


```
ZCOUNT key min max
```

* 示例：


```bash
127.0.0.1:6379> ZCOUNT scoreboard 80 90   
(integer) 1   
```

##### ZREMRANGEBYRANK

> 按照排名范围删除元素

* 语法：


```
ZREMRANGEBYRANK key start stop
```

* 示例：


```bash
127.0.0.1:6379> ZREMRANGEBYRANK scoreboard 0 1
(integer) 2 
127.0.0.1:6379> ZRANGE scoreboard 0 -1
1) "lisi"
```

##### ZREMRANGEBYSCORE

> 按照分数范围删除元素 

* 语法：


```
ZREMRANGEBYSCORE key min max
```

* 示例：


```bash
127.0.0.1:6379> zadd scoreboard 84 zhangsan	
(integer) 1
127.0.0.1:6379> ZREMRANGEBYSCORE scoreboard 80 100
(integer) 1
```

##### ZRANK/ZREVRANK

>获取元素的排名。
> - ZRANK：从小到大
> - ZREVRANK：从大到小

* 语法：


```
ZRANK key member
ZREVRANK key member
```

* 示例：


```bash
127.0.0.1:6379> ZRANK scoreboard lisi 
(integer) 0
127.0.0.1:6379> ZREVRANK scoreboard zhangsan 
(integer) 1
```

### 应用之商品销售排行榜

* 需求：根据商品销售量对商品进行排行显示

* 设计：定义商品销售排行榜（sorted set集合），Key为items:sellsort，分数为商品销售量。

* 商品编号`1001`的销量是`9`，商品编号`1002`的销量是`10`

```bash
192.168.101.3:7007>ZADD items:sellsort 9 1001 10 1002   
```

* 商品编号`1001`的销量加`1`

```bash
192.168.101.3:7001>ZINCRBY items:sellsort 1 1001   
```

* 商品销量前`10`名：

```bash
192.168.101.3:7001>ZRANGE items:sellsort 0 9 withscores   
```



## 通用命令

> keys、del、exists、expire、rename、type

### keys

> 返回满足给定pattern 的所有key

* 语法：


```
keys pattern
```

* 示例：


```bash
redis 127.0.0.1:6379> keys mylist*
1) "mylist"
2) "mylist5"
3) "mylist6"
4) "mylist7"
5) "mylist8"
```

### del

* 语法：


```
DEL key
```

* 示例：


```bash
127.0.0.1:6379> del test   
(integer)   1   
```

### exists

> 确认一个key 是否存在

* 语法：


```
exists key
```

* 示例：从结果来看，数据库中不存在`HongWan` 这个`key`，但是`age` 这个`key` 是存在的


```bash
redis 127.0.0.1:6379> exists HongWan
(integer) 0
redis 127.0.0.1:6379> exists age
(integer) 1
redis 127.0.0.1:6379>
```

### expire

> Redis在实际使用过程中更多的用作缓存，然而缓存的数据一般都是需要设置生存时间的，即：到期后数据销毁。

* 语法：


```bash
EXPIRE key seconds     #设置key的生存时间（单位：秒）key在多少秒后会自动删除   
TTL key                #查看key生于的生存时间   
PERSIST key            #清除生存时间      
PEXPIRE key milliseconds #生存时间设置单位为：毫秒   
```

示例：

```bash
192.168.101.3:7002> set test 1		    #设置test的值为1
OK
192.168.101.3:7002> get test			#获取test的值
"1"
192.168.101.3:7002> EXPIRE test 5	   #设置test的生存时间为5秒
(integer) 1
192.168.101.3:7002> TTL test			#查看test的生于生成时间还有1秒删除
(integer) 1
192.168.101.3:7002> TTL test
(integer) -2
192.168.101.3:7002> get test			#获取test的值，已经删除
(nil)
```

### rename

> 重命名key

* 语法：


```
rename  oldkey  newkey
```

* 示例：`age` 成功的被我们改名为`age_new` 了


```bash
redis 127.0.0.1:6379[1]> keys *
1) "age"
redis 127.0.0.1:6379[1]> rename age age_new
OK
redis 127.0.0.1:6379[1]> keys *
1) "age_new"
redis 127.0.0.1:6379[1]>
```

### type

> 显示指定key的数据类型

* 语法：


```
type key
```

* 示例：这个方法可以非常简单的判断出值的类型


```bash
redis 127.0.0.1:6379> type addr
string
redis 127.0.0.1:6379> type myzset2
zset
redis 127.0.0.1:6379> type mylist
list
redis 127.0.0.1:6379>
```

# Redis事务

## Redis事务介绍

* `Redis`的事务是通过`MULTI`、`EXEC`、`DISCARD`和`WATCH`这四个命令来完成的。
* `Redis`的单个命令本身都是原子性的，redis中的事务主要是用来管理**redis命令集合**。
* `Redis`将命令集合序列化并确保处于同一事务的**命令集合连续且不被打断**的执行
* `Redis`不支持回滚操作
  * 回滚是比较耗费性能的。
  * redis出现问题，都是<u>语法问题</u>和<u>类型问题</u>，这些问题，都是在编码阶段必须要解决的问题。所以运行时不会出问题。

## 事务命令

> multi（开启事务）、exec（提交事务）、discard（清除事务队列）、watch（监控）

![redis事务理解](https://ws4.sinaimg.cn/large/006tNbRwgy1fys9whcx2xj30ye0eljrx.jpg)

### MULTI

> 用于标记事务块的开始——即**开启事务**！
> Redis会将后续的命令逐个**放入队列**中，然后使用`EXEC`命令原子化地执行这个命令序列(队列中命令按序依次执行)。——如果没有开启事务，那么从客户端发送过来的命令就立刻执行；若开启事务，那么从客户端发送过来的命令会事先存入到一个事务队列中，先不执行，待执行exec命令后，事务队列中命令则按序依次执行。

* 语法：

```
multi
```

### EXEC

> 在一个事务中执行所有先前放入队列的命令，然后恢复正常的连接状态——即**提交事务**

* 语法：


```
exec
```

### DISCARD

> 清除所有先前在一个事务中放入队列的命令，然后恢复正常的连接状态（清除最近的multi开启但未exec的事务队列，期间放入队列中的命令自然也清除掉了）——即**清除当前未exec事务**

* 语法：


```
discard
```

### WATCH

> 当某个**[事务需要按条件执行]**时，就要使用这个命令将给定的**[键设置为受监控]**的状态。——监控一个key发生变化，那么exec的事务队列在key发生变化后的操作无效——即**监控（乐观锁式实现）**

* 语法：


```
watch key [key…]
```

**补充：**使用该命令可以实现`Redis`的**乐观锁**。

### UNWATCH

> 清除所有先前为一个事务监控的键。

* 语法：


```
unwatch
```

## 事务演示

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set s1 111
QUEUED
127.0.0.1:6379> hset set1 name zhangsan
QUEUED
127.0.0.1:6379> exec
1) OK
2) (integer) 1
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set s2 222
QUEUED
127.0.0.1:6379> hset set2 age 20
QUEUED
127.0.0.1:6379> discard
OK
127.0.0.1:6379> exec
(error) ERR EXEC without MULTI

127.0.0.1:6379> watch s1    
OK
127.0.0.1:6379> multi 
OK
127.0.0.1:6379> set s1 555
QUEUED
127.0.0.1:6379> exec 		 # 此时在没有exec之前，通过另一个命令窗口对监控的s1字段进行修改
(nil)
127.0.0.1:6379> get s1
111
```

## 事务失败处理

* `Redis`语法错误

![1545819902821](https://ws2.sinaimg.cn/large/006tNc79gy1fze945xtdbj30fe04tq4c.jpg)                                                  

* `Redis`运行错误

![1545819898033](https://ws4.sinaimg.cn/large/006tNc79gy1fze94hy7l6j30fe049gmd.jpg)   

* 错误总结：

  * 类型错误，原子操作都能进入队列，进入队列的命令且在类型错误之前的命令会执行成功；
  * 语法错误，错误操作原子操作不进入队列，自然均不会成功执行；

* `Redis`不支持事务回滚（为什么呢）

  1、大多数事务失败是因为**语法错误或者类型错误**，这两种错误，在开发阶段都是可以预见的

  2、`Redis`为了**性能方面**就忽略了事务回滚。

# Redis实现分布式锁

## 锁的处理

* 单应用中使用锁：（单进程多线程）

  ```
  synchronize、ReentrantLock
  ```

* 分布式应用中使用锁：（多进程多线程）

  ```
  分布式锁是控制分布式系统之间同步访问共享资源的一种方式。
  ```


## 分布式锁的实现方式

* 基于数据库的乐观锁实现分布式锁

* 基于`zookeeper`临时节点的分布式锁

  ![img](https://ws4.sinaimg.cn/large/006tNc79gy1fze95joj1cj30hs0iq756.jpg)

* 基于`Redis`的分布式锁

## 分布式锁的注意事项

* **互斥性：**在任意时刻，只有一个客户端能持有锁

* **同一性：**加锁和解锁必须是同一个客户端，客户端自己不能把别人加的锁给解了。

* **可重入性：**即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。

## 实现分布式锁

### 获取锁

![1545819972226](https://ws2.sinaimg.cn/large/006tNc79gy1fze978od7wj30fg056q4b.jpg)                                                          

#### 方式1（**使用set命令实现**）--推荐

```java
/**
	 * 使用redis的set命令实现获取分布式锁
	 * @param lockKey   	可以就是锁
	 * @param requestId		请求ID，保证同一性
	 * @param expireTime	过期时间，避免死锁
	 * @return
	 */
	public static boolean getLock(String lockKey,String requestId,int expireTime) {
		//NX:保证互斥性
		String result = jedis.set(lockKey, requestId, "NX", "EX", expireTime);
		if("OK".equals(result)) {
			return true;
		}
		
		return false;
	}

```

#### 方式2（**使用setnx命令实现**）

```java
   public static boolean getLock(String lockKey,String requestId,int expireTime) {
		Long result = jedis.setnx(lockKey, requestId);
		if(result == 1) {
			jedis.expire(lockKey, expireTime);
			return true;
		}
		
		return false;
	}
```

### 释放锁

#### 方式1（del命令实现）

```java
	/**
	 * 释放分布式锁
	 * @param lockKey
	 * @param requestId
	 */
	public static void releaseLock(String lockKey,String requestId) {
	    if (requestId.equals(jedis.get(lockKey))) {
	        jedis.del(lockKey);
	    }
	}

```

#### 方式2（**redis+lua脚本实现**）--推荐

```java
	public static boolean releaseLock(String lockKey, String requestId) {
		String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
		Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(requestId));

		if (result.equals(1L)) {
			return true;
		}
		return false;
	}
```

 

# Redis持久化

`Redis`是一个**内存**数据库，为了保证数据的持久性，它提供了两种持久化方案：

- `RDB`方式（默认）
- `AOF`方式

## RDB方式

### RDB介绍

`RDB`是`Redis`**默认**采用的持久化方式。

`RDB`方式是通过**快照**（`snapshotting`）完成的，当**符合一定条件**时`Redis`会自动将内存中的数据进行快照并持久化到硬盘。

`Redis`会在**指定的情况**下触发快照：

```
1.  符合自定义配置的快照规则
2.  执行save或者bgsave命令
3.  执行flushall命令
4.  执行主从复制操作
```

**特别说明：**

* `Redis`启动后会读取`RDB`快照文件，将数据从硬盘载入到内存。

* 根据数据量大小与结构和服务器性能不同，这个快照条件也需要不同。通常将记录**一千万个字符串类型键**、大小为`1GB`的快照文件载入到内存中需要花费20～30秒钟。

### 设置快照规则

> redis.conf（条件是以save开头的行，一行是一个条件，条件之间是或的关系）

示例：

```bash
save 900 1
save 300 10
save 60 10000

dir ./

dbfilename dump.rdb
```

**1.  RDB持久化条件**

**格式：**

```bash
save <seconds> <changes>
```

**示例：可以配置多个条件（每行配置一个条件），每个条件之间是“或”的关系**

```bash
save 900 1  #表示15分钟（900秒钟）内至少1个键被更改则进行快照。
save 300 10 #表示5分钟（300秒）内至少10个键被更改则进行快照。
save 60 10000 #表示1分钟内至少10000个键被更改则进行快照。
```

**补充**：不开启RDB方式`redis.conf`

```bash
save ""
```

**2.  配置dir指定rdb快照文件的位置**

```properties
# Note that you must specify a directory here, not a file name.   
dir   ./   
```

**3.**  **配置dbfilename指定rdb快照文件的名称**

```properties
# The filename where to dump the DB
dbfilename dump.rdb
```

### RDB快照的实现原理

**快照过程**

1. `Redis`调用系统中的`fork`函数**复制**一份当前进程的**副本**(子进程)

2. **父进程**继续接收并处理客户端发来的命令，而**子进程**开始将内存中的数据写入硬盘中的**临时文件**。

3. 当**子进程**写入完所有数据后会用该**临时文件**替换**旧的`RDB`文件**，至此，一次快照操作完成。  

![1545820028059](https://ws4.sinaimg.cn/large/006tNbRwgy1fys3a57qoej30fg06kq3v.jpg)                                                          

**注意事项**

1. `Redis`在进行快照的过程中不会修改`RDB`文件，只有快照结束后才会将旧的文件**直接替换成新的**，也就是说任何时候`RDB`文件都是完整的。 

2. 这就使得我们可以通过定时备份`RDB`文件来实现`Redis`数据库的备份， `RDB`文件是`经过压缩的二进制文件`，占用的空间会小于内存中的数据，更加利于传输。

### RDB优缺点

* **缺点：**<u>有可能发生数据丢失</u>。使用`RDB`方式实现持久化，一旦`Redis`**异常退出**，就会**丢失最后一次快照以后更改的所有数据**。这个时候我们就需要根据具体的应用场景，通过组合设置自动快照条件的方式来将可能发生的数据损失控制在能够接受范围。如果数据相对来说比较重要，希望将损失降到最小，则可以使用**`AOF`**方式进行持久化

* **优点：** `RDB`可以最大化`Redis`的性能：父进程在保存`RDB`文件时唯一要做的就是`fork`出一个子进程，然后这个子进程就会处理接下来的所有保存工作（fork一次一直延用），父进程无需执行任何磁盘`I/O`操作。同时这个也是一个缺点，如果数据集比较大的时候，`fork`可以能比较耗时，造成服务器在一段时间内停止处理客户端的请求；

## AOF方式

### AOF介绍

* 默认情况下`Redis`没有开启`AOF`（`append only file`）方式的持久化。——需要手动开启


* 开启`AOF`持久化后，每执行一条会**更改`Redis`中的数据的命令**，`Redis`就会将该命令写入硬盘中的`AOF`文件（⚠️：AOF文件记录的是数据的操作命令），这一过程显然**会降低`Redis`的性能**，但大部分情况下这个影响是能够接受的，另外使**用较快的硬盘可以提高`AOF`的性能**。




**示例**`redis.conf`：

```properties
# 可以通过修改redis.conf配置文件中的appendonly参数开启
appendonly yes   

# AOF文件的保存位置和RDB文件的位置相同，都是通过dir参数设置的。
dir   ./   

# 默认的文件名是appendonly.aof，可以通过appendfilename参数修改
appendfilename   appendonly.aof
```

### 同步磁盘数据

`Redis`每次更改数据的时候， `aof`机制都会将命令记录到`aof`文件，但是实际上由于操作系统的缓存机制，数据并没有实时写入到硬盘，而是进入**硬盘缓存**。再通过**硬盘缓存机制**去刷新到保存到文件(保存的命令)。

![](https://ws1.sinaimg.cn/large/006tNc79gy1fythitjyunj31am0oc43z.jpg)

**参数说明：**

```properties
# 每次执行写入都会进行同步， 这个是最安全但是是效率比较低的方式
appendfsync always  

# 每一秒执行(默认)
appendfsync everysec    

# 不主动进行同步操作，由操作系统去执行，这个是最快但是最不安全的方式
appendfsync no  		
```

### AOF重写原理（优化AOF文件）

1. `Redis` 可以在 `AOF` **文件体积变得过大**时，自动地在后台对 `AOF` **进行重写**。重写后的新 `AOF` 文件包含了恢复当前数据集所需的**最小命令集合**。 
2. **整个重写操作是绝对安全的**，因为 `Redis` 在创建**新 `AOF` 文件**的过程中，会继续将命令追加到**现有的 `AOF` 文件**里面，即使重写过程中发生停机，现有的 `AOF` 文件也不会丢失。 而一旦**新 `AOF` 文件**创建完毕，`Redis` 就会从**旧 `AOF` 文件**切换到**新 `AOF` 文件**，并开始对**新 `AOF` 文件**进行追加操作。
3. `AOF` 文件有序地保存了对数据库执行的所有写入操作， 这些写入操作以 `Redis` 协议的格式保存， 因此 `AOF` 文件的内容非常容易被人读懂， 对文件进行分析（`parse`）也很轻松。

   > 重写流程：
   >
   > 1. 启动一个新的进程去负责重写，先将当前aof文件中的命令进行整理，找到可以恢复当前数据的最小命令集合，存储到一个新的文件中；
   > 2. 当新文件写完，才会将旧的aof文件替换掉。
   >    1. 如果新文件写入过程中，失败，则依然使用旧文件；
   >    2. 如果新文件写入过程中，有新操作依然往旧文件中去写，然后当新文件写完后，会将新命令部分操作进行追加

**参数说明：**

```properties
# 表示当前aof文件大小超过上一次aof文件大小的百分之多少的时候会进行重写。如果之前没有重写过，以启动时aof文件大小为准
auto-aof-rewrite-percentage 100

# 限制允许重写最小aof文件大小，也就是文件大小小于64mb的时候，不需要进行优化
auto-aof-rewrite-min-size 64mb
```

### AOF文件损坏以后如何修复

* **问题描述：**

```
服务器可能在程序正在对 `AOF` 文件进行写入时停机， 如果停机造成了 `AOF` 文件出错（`corrupt`）， 那么 `Redis` 在重启时会拒绝载入这个 `AOF` 文件， 从而确保数据的一致性不会被破坏。
```

* **当发生这种情况时， 可以用以下方法来修复出错的 `AOF` 文件：**

  * 为现有的 `AOF` 文件创建一个**备份**。

  * 使用 `Redis` 附带的 **`redis-check-aof`** 程序，对原来的 `AOF` 文件进行修复。

    ```bash
    redis-check-aof --fix readonly.aof
    ```

  * 重启 `Redis` 服务器，等待服务器载入修复后的 `AOF` 文件，并进行数据恢复。

## RDB和AOF选择

* 一般来说,如果对数据的**安全性要求非常高**的话，应该**同时使用两种持久化功能**。

* 如果可以承受**数分钟以内的数据丢失**，那么可以**只使用 `RDB`** 持久化。

* 有很多用户都只使用 `AOF` 持久化， 但并不推荐这种方式：

  *  因为定时生成 `RDB` 快照（`snapshot`）非常便于进行数据库备份
  * 并且 `RDB` 恢复数据集的速度也要比 `AOF` 恢复的速度要快 。

  ```bash
  # 禁止RDB方式
  save ""
  ```


* 两种持久化策略可以同时使用，也可以使用其中一种。如果同时使用的话， 那么`Redis`重启时，会优先**使用`AOF`文件来**还原数据。

# Redis主从复制

## 什么是主从复制

持久化保证了即使`Redis`服务重启也不会丢失数据，因为`Redis`服务重启后会将硬盘上持久化的数据恢复到内存中，但是当`Redis`服务器的硬盘损坏了可能会导致数据丢失，不过**通过`Redis`的主从复制机制就可以避免这种单点故障**，如下图：

![1545997054554](https://ws1.sinaimg.cn/large/006tNc79gy1fyti4thglaj30eh097glr.jpg)



**说明：**

- 主`Redis`对外提供读写功能；从`Redis`可提供读操作，也可不提供，但绝不提供写操作，它的数据来源只能是从它的主`Redis`同步而来；

- 主`Redis`中的数据有两个副本（`replication`），即从`redis1`和从`redis2`，即使一台`Redis`服务器宕机其它两台`Redis`服务也可以继续提供服务。

- 主`Redis`中的数据和从`Redis`上的数据保持实时同步，当主`Redis`写入数据时通过**主从复制机制**会复制到两个从`Redis`服务上。

- 只有一个主`Redis`，可以有多个从`Redis`。

- 主从复制不会阻塞`master`，在同步数据时，`master` 可以继续处理`client` 请求。

- 一个`Redis`可以即是主又是从，如下图：

  ![1545997113980](https://ws2.sinaimg.cn/large/006tNc79gy1fyti5dexrej30g20eh3yw.jpg)

## 主从配置

### 主Redis配置

无需特殊配置

### 从Redis配置


修改从服务器上的`redis.conf`文件：

```properties
# slaveof <masterip> <masterport>
# 表示当前【从服务器】对应的【主服务器】的IP是192.168.10.135，端口是6379。
slaveof 192.168.10.135 6379
```

## 实现原理

* `Redis`的主从同步，分为**全量同步**和**增量同步**。

* 只有从机第一次连接上主机是**全量同步**。

* 断线重连有可能触发**全量同步**也有可能是**增量同步**（`master`判断`runid`是否一致）。

* 除此之外的情况都是**增量同步**。

![1545997494689](https://ws4.sinaimg.cn/large/006tNc79gy1fytiry5eexj30a10btglu.jpg)

### 全量同步

**`Redis`的全量同步过程主要分三个阶段：**

* **同步快照阶段：**`Master`创建并发送**快照**给`Slave`，`Slave`载入并解析快照。`Master`同时将此阶段所产生的新的写命令存储到缓冲区。

* **同步写缓冲阶段：**`Master`向`Slave`同步存储在缓冲区的写操作命令。

* **同步增量阶段：**`Master`向`Slave`同步写操作命令。

![1545997505485](https://ws2.sinaimg.cn/large/006tNc79gy1fytisa8uf2j30fg0jijt4.jpg)



### 增量同步

* `Redis`增量同步主要指**`Slave`完成初始化后开始正常工作时，`Master`发生的写操作同步到`Slave`的过程**。

* 通常情况下，`Master`每执行一个写命令就会向`Slave`发送相同的**写命令**，然后`Slave`接收并执行。



# Redis哨兵机制

`Redis`主从复制的缺点：没有办法对`master`进行动态选举，需要使用**`Sentinel`机制**完成动态选举。

## 简介

* `Sentinel`(哨兵)进程是用于**监控`Redis`集群中`Master`主服务器工作的状态**

* 在`Master`主服务器发生故障的时候，可以实现`Master`和`Slave`服务器的切换，保证系统的**高可用（`HA`）**

* 其已经被集成在`redis2.6+`的版本中，`Redis`的哨兵模式到了`2.8`版本之后就稳定了下来。

## 哨兵机制作用

- **监控(`Monitoring`):** 哨兵(`sentinel`) 会不断地检查`Master`和`Slave`是否运作正常。
- **提醒(`Notification`)**： 当被监控的某个`Redis`节点出现问题时, 哨兵(`sentinel`) 可以通过 `API` 向管理员或者其他应用程序发送通知。
- **自动故障迁移(`Automatic failover`)**：当**一个`Master`不能正常工作**时，哨兵(`sentinel`) 会开始**一次自动故障迁移操作**。
  - 它会将失效`Master`的其中一个`Slave`升级为新的`Master`, 并让失效`Master`的其他`Slave`改为复制新的`Master`；
  - 当客户端试图连接失效的`Master`时，集群也会向客户端返回新`Master`的地址，使得集群可以使用现在的`Master`替换失效`Master`。
  - `Master`和`Slave`服务器切换后，`Master`的`redis.conf`、`Slave`的`redis.conf`和`sentinel.conf`的配置文件的内容都会发生相应的改变，即，`Master`主服务器的`redis.conf`配置文件中会多一行`slaveof`的配置，`sentinel.conf`的监控目标会随之调换。

## 哨兵机制原理

1. 每个`Sentinel`（哨兵）进程以**每秒钟一次**的频率向整个集群中的`Master`主服务器，`Slave`从服务器以及其他`Sentinel`（哨兵）进程发送一个 `PING` 命令。

2. 如果一个实例（`instance`）距离最后一次有效回复 `PING` 命令的时间超过 `down-after-milliseconds` 选项所指定的值， 则这个实例会被 `Sentinel`（哨兵）进程标记为**主观下线**（**`SDOWN`**）。

3. 如果一个`Master`主服务器被标记为主观下线（`SDOWN`），则正在监视这个`Master`主服务器的**所有 `Sentinel`（哨兵）**进程要以每秒一次的频率**确认`Master`主服务器**的确**进入了主观下线状态**。

4. 当**有足够数量的 `Sentinel`（哨兵）**进程（大于等于配置文件指定的值）在指定的时间范围内确认`Master`主服务器进入了主观下线状态（`SDOWN`）， 则`Master`主服务器会被标记为**客观下线（`ODOWN`）**。

5. 在一般情况下， 每个 `Sentinel`（哨兵）进程会以每 `10` 秒一次的频率向集群中的所有`Master`主服务器、`Slave`从服务器发送 `INFO` 命令。

6. 当`Master`主服务器被 `Sentinel`（哨兵）进程标记为**客观下线（`ODOWN`）**时，`Sentinel`（哨兵）进程向下线的 `Master`主服务器的所有 `Slave`从服务器发送 `INFO` 命令的频率会从 `10` 秒一次改为每秒一次。

7. 若没有足够数量的 `Sentinel`（哨兵）进程同意 `Master`主服务器下线， `Master`主服务器的客观下线状态就会被移除。若 `Master`主服务器重新向 `Sentinel`（哨兵）进程发送 `PING` 命令返回有效回复，`Master`主服务器的主观下线状态就会被移除。

 ![1545998235717](https://ws2.sinaimg.cn/large/006tNc79gy1fytj4s6gulj30fe06t0to.jpg)                                                 

## 案例演示

* 修改从机的**`sentinel.conf`**：

  ```properties
  #sentinel monitor <master-name> <master ip> <master port> <quorum>
  sentinel monitor mymaster 192.168.10.133 6379 1
  ```


* **其他配置项说明**

  ```properties
  # Example sentinel.conf
   
  # 哨兵sentinel实例运行的端口 默认26379
  port 26379
   
  # 哨兵sentinel的工作目录
  dir /tmp
   
  # 哨兵sentinel监控的redis主节点的 ip port 
  # master-name  可以自己命名的主节点名字 只能由字母A-z、数字0-9 、这三个字符".-_"组成。
  # quorum 当这些quorum个数sentinel哨兵认为master主节点失联 那么这时 客观上认为主节点失联了
  # sentinel monitor <master-name> <ip> <redis-port> <quorum>
  sentinel monitor mymaster 127.0.0.1 6379 2
   
  # 当在Redis实例中开启了requirepass foobared 授权密码 这样所有连接Redis实例的客户端都要提供密码
  # 设置哨兵sentinel 连接主从的密码 注意必须为主从设置一样的验证密码
  # sentinel auth-pass <master-name> <password>
  sentinel auth-pass mymaster MySUPER--secret-0123passw0rd
   
   
  # 指定多少毫秒之后 主节点没有应答哨兵sentinel 此时 哨兵主观上认为主节点下线 默认30秒
  # sentinel down-after-milliseconds <master-name> <milliseconds>
  sentinel down-after-milliseconds mymaster 30000
   
  # 这个配置项指定了在发生故障转移failover主备切换时最多可以有多少个slave同时对新的master进行同步，这个数字越小，完成failover所需的时间就越长，但是如果这个数字越大，就意味着越多的slave因为replication而不可用。可以通过将这个值设为 1 来保证每次只有一个slave 处于不能处理命令请求的状态。
  # sentinel parallel-syncs <master-name> <numslaves>
  sentinel parallel-syncs mymaster 1
   
   
   
  # 故障转移的超时时间 failover-timeout 可以用在以下这些方面： 
  #1. 同一个sentinel对同一个master两次failover之间的间隔时间。
  #2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。
  #3.当想要取消一个正在进行的failover所需要的时间。  
  #4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来了
  # 默认三分钟
  # sentinel failover-timeout <master-name> <milliseconds>
  sentinel failover-timeout mymaster 180000
  
  # SCRIPTS EXECUTION
   
  #配置当某一事件发生时所需要执行的脚本，可以通过脚本来通知管理员，例如当系统运行不正常时发邮件通知相关人员。
  #对于脚本的运行结果有以下规则：
  #若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10
  #若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。
  #如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。
  #一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。
   
  #通知型脚本:当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等），将会去调用这个脚本，这时这个脚本应该通过邮件，SMS等方式去通知系统管理员关于系统不正常运行的信息。调用该脚本时，将传给脚本两个参数，一个是事件的类型，一个是事件的描述。如果sentinel.conf配置文件中配置了这个脚本路径，那么必须保证这个脚本存在于这个路径，并且是可执行的，否则sentinel无法正常启动成功。
  #通知脚本
  # sentinel notification-script <master-name> <script-path>
    sentinel notification-script mymaster /var/redis/notify.sh
    
  # 客户端重新配置主节点参数脚本
  # 当一个master由于failover而发生改变时，这个脚本将会被调用，通知相关的客户端关于master地址已经发生改变的信息。
  # 以下参数将会在调用脚本时传给脚本:
  # <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
  # 目前<state>总是“failover”,
  # <role>是“leader”或者“observer”中的一个。 
  # 参数 from-ip, from-port, to-ip, to-port是用来和旧的master和新的master(即旧的slave)通信的
  # 这个脚本应该是通用的，能被多次调用，不是针对性的。
  # sentinel client-reconfig-script <master-name> <script-path>
   sentinel client-reconfig-script mymaster /var/redis/reconfig.sh
  ```

* 通过`redis-sentinel`启动哨兵服务

  ```bash
  ./redis-sentinel sentinel.conf
  ```



# Redis集群

redis3.0以后推出的redis cluster 集群方案，redis cluster集群保证了**高可用**、**高性能**、**高可扩展性**。

## Redis的集群策略

l  推特：twemproxy

l  豌豆荚：codis

l  官方：redis cluster

## Redis-cluster架构图

 ![1546432188097](https://ws4.sinaimg.cn/large/006tNc79gy1fzed63vosvj30dt0fwwji.jpg)                                                         

架构细节:

(1)所有的redis节点（master节点）彼此互联(**PING-PONG机制**),内部使用**二进制协议**优化传输速度和带宽.

(2)节点的fail是通过集群中超过半数的节点检测失效时才生效.

(3)客户端与redis节点直连,不需要中间proxy层.客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可

(4)redis-cluster把所有的物理节点映射到[0-16383]**slot(槽)**上,cluster 负责维护**node(节点)<->slot(槽)<->value(数据)**

```
Redis集群中内置了16384个哈希槽，当需要在Redis集群中放置一个key-value时，redis 先对 key 使用 crc16 算法算出一个结果，然后把结果对 16384 求余数，这样每个 key 都会对应一个编号在 0-16383 之间的哈希槽，redis 会根据节点数量大致均等的将哈希槽映射到不同的节点
```

示例如下：

![1546432204175](https://ws4.sinaimg.cn/large/006tNc79gy1fzf45fj10lj30hx0hedg9.jpg)               

​          Server1     0-5000          

## Redis-cluster投票:容错

![1546432229084](https://ws2.sinaimg.cn/large/006tKfTcgy1g05r6if71nj30fg0cbjw9.jpg)   

最小节点数：**3台**

(1)**节点失效判断**：集群中所有master参与投票,如果**半数以上master节点**与其中一个master节点通信超过**(cluster-node-timeout)**,认为该master节点挂掉.

(2)**集群失效判断原则**［什么时候整个集群不可用(cluster_state:fail)? ］

* 如果集群任意master挂掉,且当前master没有slave，则集群进入fail状态。也可以理解成集群的[0-16383]slot映射不完全时进入fail状态。
* 如果集群超过半数以上master挂掉，无论是否有slave，集群进入fail状态。

## 安装Ruby环境

redis集群需要使用**集群管理脚本redis-trib.rb**，它的执行相应依赖ruby环境。

l  第一步：安装ruby

```
yum   install ruby   
yum   install rubygems   
```

l  第二步：安装ruby和redis的接口程序redis-3.2.2.gem

```
gem   install redis -V 3.2.2     
```

l  第三步：复制redis-3.2.9/src/redis-trib.rb文件到/usr/local/redis目录

```
cp   redis-3.2.9/src/redis-trib.rb /usr/local/redis-cluster/ -r    
```

## 安装RedisCluster

```properties
Redis集群最少需要三台主服务器，三台从服务器(三主三从)。
端口号分别为：7001~7006
```

* 第一步：创建7001实例，并编辑redis.conf文件，修改port为7001。

  注意：创建实例，即拷贝单机版安装时，生成的bin目录，端口为7001。

  ![1546440875268](https://ws3.sinaimg.cn/large/006tNc79gy1fzfbtmoq1jj30fe0223ys.jpg)   

* 第二步：修改redis.conf配置文件，打开Cluster-enable yes

  ![1546440886541](https://ws3.sinaimg.cn/large/006tNc79gy1fzfbtxz35gj30fd06edi7.jpg)

* 第三步：复制7001，创建7002~7006实例，**注意端口修改**。

* 第四步：启动所有的实例

* 第五步：创建Redis集群

  ```bash
  ./redis-trib.rb create --replicas 1 111.231.106.221:7001 111.231.106.221:7002 111.231.106.221:7003 111.231.106.221:7004 111.231.106.221:7005  111.231.106.221:7006
  >>> Creating cluster
  Connecting to node 192.168.10.133:7001: OK
  Connecting to node 192.168.10.133:7002: OK
  Connecting to node 192.168.10.133:7003: OK
  Connecting to node 192.168.10.133:7004: OK
  Connecting to node 192.168.10.133:7005: OK
  Connecting to node 192.168.10.133:7006: OK
  >>> Performing hash slots allocation on 6 nodes...
  Using 3 masters:
  192.168.10.133:7001
  192.168.10.133:7002
  192.168.10.133:7003
  Adding replica 192.168.10.133:7004 to 192.168.10.133:7001
  Adding replica 192.168.10.133:7005 to 192.168.10.133:7002
  Adding replica 192.168.10.133:7006 to 192.168.10.133:7003
  M: d8f6a0e3192c905f0aad411946f3ef9305350420 192.168.10.133:7001
     slots:0-5460 (5461 slots) master
  M: 7a12bc730ddc939c84a156f276c446c28acf798c 192.168.10.133:7002
     slots:5461-10922 (5462 slots) master
  M: 93f73d2424a796657948c660928b71edd3db881f 192.168.10.133:7003
     slots:10923-16383 (5461 slots) master
  S: f79802d3da6b58ef6f9f30c903db7b2f79664e61 192.168.10.133:7004
     replicates d8f6a0e3192c905f0aad411946f3ef9305350420
  S: 0bc78702413eb88eb6d7982833a6e040c6af05be 192.168.10.133:7005
     replicates 7a12bc730ddc939c84a156f276c446c28acf798c
  S: 4170a68ba6b7757e914056e2857bb84c5e10950e 192.168.10.133:7006
     replicates 93f73d2424a796657948c660928b71edd3db881f
  Can I set the above configuration? (type 'yes' to accept): yes
  >>> Nodes configuration updated
  >>> Assign a different config epoch to each node
  >>> Sending CLUSTER MEET messages to join the cluster
  Waiting for the cluster to join....
  >>> Performing Cluster Check (using node 192.168.10.133:7001)
  M: d8f6a0e3192c905f0aad411946f3ef9305350420 192.168.10.133:7001
     slots:0-5460 (5461 slots) master
  M: 7a12bc730ddc939c84a156f276c446c28acf798c 192.168.10.133:7002
     slots:5461-10922 (5462 slots) master
  M: 93f73d2424a796657948c660928b71edd3db881f 192.168.10.133:7003
     slots:10923-16383 (5461 slots) master
  M: f79802d3da6b58ef6f9f30c903db7b2f79664e61 192.168.10.133:7004
     slots: (0 slots) master
     replicates d8f6a0e3192c905f0aad411946f3ef9305350420
  M: 0bc78702413eb88eb6d7982833a6e040c6af05be 192.168.10.133:7005
     slots: (0 slots) master
     replicates 7a12bc730ddc939c84a156f276c446c28acf798c
  M: 4170a68ba6b7757e914056e2857bb84c5e10950e 192.168.10.133:7006
     slots: (0 slots) master
     replicates 93f73d2424a796657948c660928b71edd3db881f
  [OK] All nodes agree about slots configuration.
  >>> Check for open slots...
  >>> Check slots coverage...
  [OK] All 16384 slots covered.
  [root@localhost-0723 redis]#
  
  ```

## 命令客户端连接集群

命令：

```
./redis-cli –h 127.0.0.1 –p 7001 –c
```

注意：**-c** 表示是以redis集群方式进行连接

```
./redis-cli -p 7006 -c
127.0.0.1:7006> set key1 123
-> Redirected to slot [9189] located at 127.0.0.1:7002
OK
127.0.0.1:7002>
```

## 查看集群状态、节点信息

Ø  查看集群状态

```
127.0.0.1:7003> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:3
cluster_stats_messages_sent:926
cluster_stats_messages_received:926

```

Ø  查看集群中的节点：

```
127.0.0.1:7003> cluster nodes
7a12bc730ddc939c84a156f276c446c28acf798c 127.0.0.1:7002 master - 0 1443601739754 2 connected 5461-10922
93f73d2424a796657948c660928b71edd3db881f 127.0.0.1:7003 myself,master - 0 0 3 connected 10923-16383
d8f6a0e3192c905f0aad411946f3ef9305350420 127.0.0.1:7001 master - 0 1443601741267 1 connected 0-5460
4170a68ba6b7757e914056e2857bb84c5e10950e 127.0.0.1:7006 slave 93f73d2424a796657948c660928b71edd3db881f 0 1443601739250 6 connected
f79802d3da6b58ef6f9f30c903db7b2f79664e61 127.0.0.1:7004 slave d8f6a0e3192c905f0aad411946f3ef9305350420 0 1443601742277 4 connected
0bc78702413eb88eb6d7982833a6e040c6af05be 127.0.0.1:7005 slave 7a12bc730ddc939c84a156f276c446c28acf798c 0 1443601740259 5 connected
127.0.0.1:7003>
```

## 维护节点

**集群创建成功后可以继续向集群中添加(主／从)节点**

### 添加主节点

* 先创建7007节点

* 添加7007结点作为新节点

  执行命令：

```
./redis-trib.rb add-node 127.0.0.1:7007 127.0.0.1:7001
```

 ![1546441118480](https://ws3.sinaimg.cn/large/006tNc79gy1fzfcrr4k5ij30fj09eaet.jpg)          

* 查看集群结点发现7007已添加到集群中

![1546441126104](https://ws2.sinaimg.cn/large/006tNc79gy1fzfcs7e359j30le02776q.jpg)           

#### hash槽重新分配（数据迁移）

**添加完主节点需要对主节点进行hash槽分配，这样该主节才可以存储数据。**

* **查看集群中槽占用情况**

```
cluster nodes
```

redis集群有16384个槽，集群中的每个结点分配自已槽，通过查看集群结点可以看到槽占用情况。

![1546441138064](https://ws3.sinaimg.cn/large/006tNc79gy1fzfcuh5h2gj30kl02qgnp.jpg)           

**给刚添加的7007结点分配槽**

##### **第一步：连接上集群（连接集群中任意一个可用结点都行）**

```
./redis-trib.rb reshard 192.168.10.133:7001
```

##### **第二步：输入要分配的槽数量**

![1546441155298](https://ws3.sinaimg.cn/large/006tNc79gy1fzfcwjra13j30fj0bj7ap.jpg)           

输入：**3000**，表示要给目标节点分配3000个槽

##### **第三步：输入接收槽的结点id**

![1546441209201](https://ws4.sinaimg.cn/large/006tNc79gy1fzfcy6twt6j30fj039q3t.jpg)           

输入：**15b809eadae88955e36bcdbb8144f61bbbaf38fb**

```
PS：这里准备给7007分配槽，通过cluster nodes查看7007结点id为：

15b809eadae88955e36bcdbb8144f61bbbaf38fb
```

##### **第四步：输入源结点id**

![1546441219997](https://ws4.sinaimg.cn/large/006tNc79gy1fzfcyqcpsqj30fj042aba.jpg)           

输入：**all**

##### **第五步：输入yes开始移动槽到目标结点id**

![1546441228798](https://ws1.sinaimg.cn/large/006tNc79gy1fzfcz5xcngj30f700tt8u.jpg)   

输入：**yes**

### 添加从节点

* **添加7008从结点，将7008作为7007的从结点**

命令：

```
./redis-trib.rb add-node --slave --master-id  主节点id   新节点的ip和端口   旧节点ip和端口（集群中任一节点都可以）
```

示例：

```
./redis-trib.rb add-node --slave --master-id  35da64607a02c9159334a19164e68dd95a3b943c 192.168.10.103:7008 192.168.10.103:7001
```

**35da64607a02c9159334a19164e68dd95a3b943c**是7007结点的id，可通过cluster nodes查看。

![1546441296230](https://ws1.sinaimg.cn/large/006tNc79gy1fzfd3y5ogyj30fj079gol.jpg) 

**注意**：如果原来该结点在集群中的配置信息已经生成到cluster-config-file指定的配置文件中（如果cluster-config-file没有指定则默认为**nodes.conf**），这时可能会报错：

```
[ERR] Node XXXXXX is not empty. Either the node already knows other nodes (check with CLUSTER NODES) or contains some key in database 0
```

**解决方法是删除生成的配置文件nodes.conf，删除后再执行./redis-trib.rb add-node指令**

* **查看集群中的结点，刚添加的7008为7007的从节点：**

![1546441323416](https://ws3.sinaimg.cn/large/006tNc79gy1fzfd6jkf6nj30j9027mzn.jpg) 

### 删除结点

示例：

```
./redis-trib.rb del-node 127.0.0.1:7005 4b45eb75c8b428fbd77ab979b85080146a9bc017
```

删除已经占有hash槽的结点会失败，报错如下：

```
[ERR] Node 127.0.0.1:7005 is not empty! Reshard data away and try again.
```

需要将该结点占用的hash槽分配出去（参考hash槽重新分配章节）。

## Jedis连接集群

需要开启防火墙，或者直接关闭防火墙。

```
service iptables stop  
```

### 代码实现

创建JedisCluster类连接redis集群。

```java
@Test
public void testJedisCluster() throws Exception {
	//创建一连接，JedisCluster对象,在系统中是单例存在
	Set<HostAndPort> nodes = new HashSet<>();
	nodes.add(new HostAndPort("192.168.10.133", 7001));
	nodes.add(new HostAndPort("192.168.10.133", 7002));
	nodes.add(new HostAndPort("192.168.10.133", 7003));
	nodes.add(new HostAndPort("192.168.10.133", 7004));
	nodes.add(new HostAndPort("192.168.10.133", 7005));
	nodes.add(new HostAndPort("192.168.10.133", 7006));
	JedisCluster cluster = new JedisCluster(nodes);
	//执行JedisCluster对象中的方法，方法和redis一一对应。
	cluster.set("cluster-test", "my jedis cluster test");
	String result = cluster.get("cluster-test");
	System.out.println(result);
	//程序结束时需要关闭JedisCluster对象
	cluster.close();
}

```

### 使用spring

#### 配置applicationContext.xml

```xml
<!-- 连接池配置 -->
<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
	<!-- 最大连接数 -->
	<property name="maxTotal" value="30" />
	<!-- 最大空闲连接数 -->
	<property name="maxIdle" value="10" />
	<!-- 每次释放连接的最大数目 -->
	<property name="numTestsPerEvictionRun" value="1024" />
	<!-- 释放连接的扫描间隔（毫秒） -->
	<property name="timeBetweenEvictionRunsMillis" value="30000" />
	<!-- 连接最小空闲时间 -->
	<property name="minEvictableIdleTimeMillis" value="1800000" />
	<!-- 连接空闲多久后释放, 当空闲时间>该值 且 空闲连接>最大空闲连接数 时直接释放 -->
	<property name="softMinEvictableIdleTimeMillis" value="10000" />
	<!-- 获取连接时的最大等待毫秒数,小于零:阻塞不确定的时间,默认-1 -->
	<property name="maxWaitMillis" value="1500" />
	<!-- 在获取连接的时候检查有效性, 默认false -->
	<property name="testOnBorrow" value="true" />
	<!-- 在空闲时检查有效性, 默认false -->
	<property name="testWhileIdle" value="true" />
	<!-- 连接耗尽时是否阻塞, false报异常,ture阻塞直到超时, 默认true -->
	<property name="blockWhenExhausted" value="false" />
</bean>
<!-- redis集群 -->
<bean id="jedisCluster" class="redis.clients.jedis.JedisCluster">
	<constructor-arg index="0">
		<set>
			<bean class="redis.clients.jedis.HostAndPort">
				<constructor-arg index="0" value="192.168.101.3"></constructor-arg>
				<constructor-arg index="1" value="7001"></constructor-arg>
			</bean>
			<bean class="redis.clients.jedis.HostAndPort">
				<constructor-arg index="0" value="192.168.101.3"></constructor-arg>
				<constructor-arg index="1" value="7002"></constructor-arg>
			</bean>
			<bean class="redis.clients.jedis.HostAndPort">
				<constructor-arg index="0" value="192.168.101.3"></constructor-arg>
				<constructor-arg index="1" value="7003"></constructor-arg>
			</bean>
			<bean class="redis.clients.jedis.HostAndPort">
				<constructor-arg index="0" value="192.168.101.3"></constructor-arg>
				<constructor-arg index="1" value="7004"></constructor-arg>
			</bean>
			<bean class="redis.clients.jedis.HostAndPort">
				<constructor-arg index="0" value="192.168.101.3"></constructor-arg>
				<constructor-arg index="1" value="7005"></constructor-arg>
			</bean>
			<bean class="redis.clients.jedis.HostAndPort">
				<constructor-arg index="0" value="192.168.101.3"></constructor-arg>
				<constructor-arg index="1" value="7006"></constructor-arg>
			</bean>
		</set>
	</constructor-arg>
	<constructor-arg index="1" ref="jedisPoolConfig"></constructor-arg>
</bean>
```

#### 测试代码

```java
private ApplicationContext applicationContext;
	@Before
	public void init() {
		applicationContext = new ClassPathXmlApplicationContext(
				"classpath:applicationContext.xml");
	}

	// redis集群
	@Test
	public void testJedisCluster() {
		JedisCluster jedisCluster = (JedisCluster) applicationContext
				.getBean("jedisCluster");

		jedisCluster.set("name", "zhangsan");
		String value = jedisCluster.get("name");
		System.out.println(value);
	}

```

# Redis和LUA整合

## 什么是lua？

lua是一种轻量小巧的**脚本语言**，用标准**C语言**编写并以源代码形式开放， 其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。

## Redis中使用lua的好处

1. **减少网络开销**，在Lua脚本中可以把多个命令放在同一个脚本中运行
2. **原子操作**，redis会将整个脚本作为一个整体执行，中间不会被其他命令插入。换句话说，编写脚本的过程中无需担心会出现竞态条件
3. **复用性**，客户端发送的脚本会永远存储在redis中，这意味着其他客户端可以复用这一脚本来完成同样的逻辑

## lua的安装（了解）

* 下载

  地址：<http://www.lua.org/download.html>

  可以本地下载上传到linux，也可以使用curl命令在linux系统中进行在线下载

  ```
  curl -R -O http://www.lua.org/ftp/lua-5.3.5.tar.gz
  ```

* 安装

```
yum -y install readline-devel ncurses-devel
tar -zxvf lua-5.3.5.tar.gz
make linux
make install
```

如果报错，说找不到readline/readline.h, 可以通过yum命令安装

```
yum -y install readline-devel ncurses-devel
```

安装完以后再

```
make linux  / make install
```

最后，直接输入 lua命令即可进入lua的控制台

## lua常见语法（了解）

详见<http://www.runoob.com/lua/lua-tutorial.html>

## Redis整合lua脚本

从Redis2.6.0版本开始，通过**内置的lua编译/解释器**，可以使用EVAL命令对lua脚本进行求值。

### EVAL命令

```
通过执行redis的eval命令，可以运行一段lua脚本。
```

* 在redis客户端中，执行以下命令：

```
EVAL script numkeys key [key ...] arg [arg ...]
```

* **命令说明：**
  * **script参数：**是一段Lua脚本程序，它会被运行在Redis服务器上下文中，这段脚本不必(也不应该)定义为一个Lua函数。
  * **numkeys参数：**用于指定键名参数的个数。
  * **key [key ...]参数：** 从EVAL的第三个参数开始算起，使用了numkeys个键（key），表示在脚本中所用到的那些Redis键(key)，这些键名参数可以在Lua中通过全局变量**KEYS**数组，用1为基址的形式访问( KEYS[1] ， KEYS[2] ，以此类推)。
  * **arg [arg ...]参数：**可以在Lua中通过全局变量**ARGV**数组访问，访问的形式和KEYS变量类似( ARGV[1] 、 ARGV[2] ，诸如此类)。
* 例如：

```lua
> eval "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
1) "key1"
2) "key2"
3) "first"
4) "second"
```

### lua脚本中调用Redis命令

* **redis.call()：**
  * 返回值就是redis命令执行的返回值
  * 如果出错，则返回错误信息，不继续执行
* **redis.pcall()：**
  * 返回值就是redis命令执行的返回值
  * 如果出错，则记录错误信息，继续执行
* **注意事项**
  * 在脚本中，使用return语句将返回值返回给客户端，如果没有return，则返回nil

* 示例：

```shell
127.0.0.1:6379> eval "return redis.call('set',KEYS[1],'bar')" 1 foo
OK
```

### EVALSHA

```
EVAL 命令要求你在每次执行脚本的时候都发送一次脚本主体(script body)。

Redis 有一个内部的缓存机制，因此它不会每次都重新编译脚本，不过在很多场合，付出无谓的带宽来传送脚本主体并不是最佳选择。

为了减少带宽的消耗， Redis 实现了 EVALSHA 命令，它的作用和 EVAL 一样，都用于对脚本求值，但它接受的第一个参数不是脚本，而是脚本的 SHA1 校验和(sum)
```

**EVALSHA命令的表现如下：**

* 如果服务器还记得给定的 SHA1 校验和所指定的脚本，那么执行这个脚本
* 如果服务器不记得给定的 SHA1 校验和所指定的脚本，那么它返回一个特殊的错误，提醒用户使用 EVAL 代替 EVALSHA

**示例**

```shell
> evalsha 6b1bf486c81ceb7edf3c093f4c48582e38c0e791 0
"bar"
```

### SCRIPT命令

* **SCRIPT FLUSH** **：**清除所有脚本缓存
* **SCRIPT EXISTS** **：**根据给定的脚本校验、检查指定的脚本是否存在于脚本缓存
* **SCRIPT LOAD** **：**将一个脚本装入脚本缓存，**返回SHA1摘要**，但并不立即运行它
* **SCRIPT KILL** **：**杀死当前正在运行的脚本

### redis-cli --eval

l  可以使用**redis-cli --eval**命令指定一个lua脚本文件去执行。

l  脚本文件(redis.lua)，内容如下：

```lua
return redis.call(‘set’,KEYS[1],ARGV[1]);
```

l  在redis客户机，执行脚本命令：

```lua
redis-cli --eval redis.lua test ,‘测试’
```

l  命令格式说明：

```
 --eval：告诉redis客户端去执行后面的lua脚本
 redis.lua：具体的lua脚本文件名称
 test : lua脚本中需要的key
 ‘测试’：lua脚本中需要的value
```

l  注意事项：

```
上面命令中keys和values中间需要使用逗号隔开，并且逗号两边都要有空格
```

# Redis消息模式

## 队列模式

使用list类型的**lpush和rpop**实现消息队列

![1546441561605](https://ws2.sinaimg.cn/large/006tNc79gy1fzfduxvdmpj30fe07fmxt.jpg)                                                  

**注意事项：**

* 消息接收方如果不知道队列中是否有消息，会一直发送rpop命令，如果这样的话，会每一次都建立一次连接，这样显然不好。
* 可以使用**brpop**命令，它如果从队列中取不出来数据，会一直阻塞，在一定范围内没有取出则返回null、

## 发布订阅模式

* 订阅消息（**subscribe**）

![1546441591888](https://ws1.sinaimg.cn/large/006tNc79gy1fzfe02xbhxj308v05iq2w.jpg)   

示例：

subscribe kkb-channel

* 发布消息（**publish**）

 ![1546441597623](https://ws3.sinaimg.cn/large/006tNc79gy1fzfe12gzhnj308r07xdfw.jpg)  

```
publish kkb-channel “我是灭霸詹”
```

* Redis发布订阅命令

![1546441605324](https://ws4.sinaimg.cn/large/006tNc79gy1fzfe1c1h8wj30fg097t9z.jpg)           

# 常见缓存问题

## 查询缓存的步骤

1. 先查询缓存，如果没有数据，再去查询数据库
2. 查询完数据库之后，如果数据不为空，再将结果写入缓存

## 缓存穿透

* 什么叫缓存穿透？

  > 一般的缓存系统，都是按照key去缓存查询，如果不存在对应的value，就应该去后端系统查找（比如DB）。如果key对应的value是一定不存在的，并且对该key并发请求量很大，就会对后端系统造成很大的压力。
  >
  > 也就是说，**==对不存在的key进行高并发访问，导致数据库压力瞬间增大，这就叫做【缓存穿透】==**。

* 如何解决？

  > 1. 对查询结果为空的情况也进行缓存，缓存时间设置短一点，或者该key对应的数据insert了之后清理缓存。
  > 2. 对一定不存在的key进行过滤。可以把所有的可能存在的key放到一个大的Bitmap中，查询时通过该bitmap过滤。（布隆表达式）

## 缓存雪崩

* 什么叫缓存雪崩？

  > ==**当缓存服务器重启或者大量缓存集中在某一个时间段失效**==，这一时间段也会给后端系统(比如DB)带来很大压力。

* 如何解决？

  > 1. 在**缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量**。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。
  > 2. **不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀**。
  > 3. 做**二级缓存**，A1为原始缓存，A2为拷贝缓存，A1失效时，可以访问A2，A1缓存失效时间设置为短期，A2设置为长期（此点为补充）

## 缓存击穿

* 什么叫缓存击穿？

  > 对于一些设置了过期时间的key，如果这些key可能会在某些时间点被超高并发地访问，是一种非常"热点"的数据。这个时候，需要考虑一个问题：缓存被"击穿"的问题，这个和缓存雪崩的区别在于这里针对某一key缓存，前者则是很多key。
  >
  > 缓存在某个时间点过期的时候，恰好在这个时间点对这个Key有大量的并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。

* 如何解决？

  > 使用**redis的setnx互斥锁**先进行判断，这样其他线程就处于等待状态，保证不会有大并发操作去操作数据库。
  >
  > if(redis.sexnx()==1){
  >    //先查询缓存
  >    //查询数据库
  >    //加入缓存
  > }

# 缓存双写一致性

一般来说，在读取缓存方面，我们都是先读取缓存，再读取数据库的。

但是，在更新缓存方面，我们是需要先更新缓存，再更新数据库？还是先更新数据库，再更新缓存？还是说有其他的方案？

## 先更新数据库再更新缓存(不建议使用)

* 操作步骤（线程A和线程B都对同一数据进行更新操作）：

```
1. 线程A更新了数据库
2. 线程B更新了数据库
3. 线程B更新了缓存
4. 线程A更新了缓存
```

* 问题1：脏读
* 问题2：**浪费性能**

## 先更新数据库再删除缓存

* 操作步骤（线程A更新、线程B读）

```
- 请求A进行写操作，删除缓存，此时A的写操作还没有执行完
- 请求B查询发现缓存不存在
- 请求B去数据库查询得到旧值
- 请求B将旧值写入缓存
- 请求A将新值写入数据库
```

 

* **解决方案1：延时双删策略，伪代码如下：**

```java
public void write(String key, Object data) {
		redis.delKey(key);
		db.updateData(data);
		Thread.sleep(1000);
		redis.deleKey(key);
	}
```

* 解决方案2：使用消息队列

## 先删除缓存再更新数据库

* 操作步骤

```
1. 用户A删除缓存失败
2. 用户A成功更新了数据
```

或者

```
1. 用户A删除了缓存；
2. 用户B读取缓存，缓存不存在；
3. 用户B从数据库拿到旧数据；
4. 用户B更新了缓存；
5. 用户A更新了数据。
```

* 问题：脏数据

* 解决方案1：设置缓存有效时间（最简单）
* 解决方案2：使用消息队列

