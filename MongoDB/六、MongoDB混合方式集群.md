## 部署图

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-06-083532.png)

## 数据服务器配置

### 副本集1的配置

在副本集中每个数据节点的mongodb.cfg配置文件【追加】以下内容( 仲裁节点除外):

```shell
# 数据库文件位置 
dbpath=/root/mongodb/datasvr/rs1/node01/data
#日志文件位置 
logpath=/root/mongodb/datasvr/rs1/node01/logs/mongodb.log 
# 以追加方式写入日志
logappend=true
# 是否以守护进程方式运行
fork = true

bind_ip=192.168.10.136
# 默认27017
port = 27003 
#注意:不需要显式的去指定主从,主从是动态选举的 
#副本集集群,需要指定一个名称,在一个副本集下,名称是相同的 
replSet=rs001
#需要分片的服务器，要设置shardsvr为true
shardsvr=true
```

```shell
# 数据库文件位置 
dbpath=/root/mongodb/datasvr/rs1/node01/data
#日志文件位置 
logpath=/root/mongodb/datasvr/rs1/node01/logs/mongodb.log 
# 以追加方式写入日志
logappend=true
# 是否以守护进程方式运行
fork = true

bind_ip=192.168.10.136
# 默认27017
port = 27004
#注意:不需要显式的去指定主从,主从是动态选举的 
#副本集集群,需要指定一个名称,在一个副本集下,名称是相同的 
replSet=rs001
#需要分片的服务器，要设置shardsvr为true
shardsvr=true
```

```shell
# 数据库文件位置 
dbpath=/root/mongodb/datasvr/rs1/node01/data
#日志文件位置 
logpath=/root/mongodb/datasvr/rs1/node01/logs/mongodb.log 
# 以追加方式写入日志
logappend=true
# 是否以守护进程方式运行
fork = true

bind_ip=192.168.10.136
# 默认27017
port = 27005
#注意:不需要显式的去指定主从,主从是动态选举的 
#副本集集群,需要指定一个名称,在一个副本集下,名称是相同的 
replSet=rs001
#需要分片的服务器，要设置shardsvr为true
shardsvr=true
```

## 配置服务器的配置

配置三个配置服务器,配置信息如下,端口和path单独指定:

```shell
# 数据库文件位置 
dbpath=/root/mongodb/bin/cluster/configsvr/node01/data
#日志文件位置 
logpath=/root/mongodb/bin/cluster/configsvr/node01/logs/mongodb.log 
# 以追加方式写入日志
logappend=true
# 是否以守护进程方式运行
fork = true
bind_ip=192.168.10.135
# 默认28001
port = 28001
# 表示是一个配置服务器
configsvr=true
#配置服务器副本集名称
replSet=configsvr
```

```shell
# 数据库文件位置 
dbpath=/root/mongodb/bin/cluster/configsvr/node02/data
#日志文件位置 
logpath=/root/mongodb/bin/cluster/configsvr/node02/logs/mongodb.log 
# 以追加方式写入日志
logappend=true
# 是否以守护进程方式运行
fork = true
bind_ip=192.168.10.135
# 默认28001
port = 28002
# 表示是一个配置服务器
configsvr=true
#配置服务器副本集名称
replSet=configsvr
```

```shell
dbpath=/root/mongodb/bin/cluster/configsvr/node03/data
#日志文件位置 
logpath=/root/mongodb/bin/cluster/configsvr/node03/logs/mongodb.log 
# 以追加方式写入日志
logappend=true
# 是否以守护进程方式运行 
fork = true 
bind_ip=192.168.10.135 
# 默认28001
port = 28003
# 表示是一个配置服务器 
configsvr=true 
#配置服务器副本集名称 
replSet=configsvr
```

注意创建dbpath和logpath

```shell
mongod -f /root/mongodb/bin/cluster/configsvr/node01/mongodb.cfg
mongod -f /root/mongodb/bin/cluster/configsvr/node02/mongodb.cfg
mongod -f /root/mongodb/bin/cluster/configsvr/node03/mongodb.cfg
```

### 配置副本集

```shell
mongo 192.168.10.135:28001
use admin
cfg={_id:"configsvr",members: [
{_id:0,host:"192.168.10.136:28001"},
{_id:1,host:"192.168.10.136:28002"},
{_id:2,host:"192.168.10.136:28003"}
]}
rs.initiate(cfg);
```

## 路由服务器配置

```shell
#1、因为路由不存数据，所以不需要datapath
#2、它要从配置服务器读取配置文件，所以它要知道配置服务器在哪儿(配置副本集名称/多个副本集IP)
configdb=configsvr/192.168.10.135:28001,192.168.10.135:28002,192.168.10.135:28003 
#日志文件位置
logpath=/root/mongodb/bin/cluster/routersvr/node01/logs/mongodb.log
# 以追加方式写入日志
logappend=true
# 是否以守护进程方式运行 
fork = true 
bind_ip=192.168.10.136 
# 默认28001
port=30000
```

路由服务器启动(注意这里是mongos命令而不是mongod命令)

```shell
mongos -f /root/mongodb/bin/cluster/routersvr/node01/mongodb.cfg
```

### 关联切片和路由

登录到路由服务器中,执行关联切片和路由的相关操作。

```shell
mongo 192.168.10.136:30000

#查看shard相关的命令 
sh.help()
```

```shell
sh.addShard("切片名称/地址") 
sh.addShard("rs001/192.168.10.136:27003");#将副本集(任一个节点即可)添加到切片中，让路由服务器知道
sh.addShard("rs002/192.168.10.136:27006");

use kkb
sh.enableSharding("kkb");#指定哪个数据库、集合进行分片
sh.shardCollection("kkb.mycollection",{name:"hashed"});#分片规则，表示按name进行分片，将对name进行hash处理，hashed是内置的字符串，将相同的name名称计算一个hash值，然后将值映射到对应的块上
for(var i=1;i<=100;i++) db.mycollection.insert({name:"James"+i,age:i});#js脚本
```

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-06-084529.png)