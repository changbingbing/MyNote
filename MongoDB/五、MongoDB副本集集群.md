## [MongoDB 复制（副本集）](https://www.runoob.com/mongodb/mongodb-replication.html)

* 优点：主如果宕机，仲裁节点会选举从作为新的主
* 缺点：一旦仲裁节点挂了，集群就废了(主从沦为单点，不再进行主从选举)
  ![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-06-082014.png)

## 节点1配置

```shell
# 数据库文件位置 
dbpath=/root/mongodb/rs/rs01/node01/data
#日志文件位置 
logpath=/root/mongodb/rs/rs01/node01/logs/mongodb.log
# 以追加方式写入日志 
logappend=true
# 是否以守护进程方式运行 
fork = true
bind_ip=192.168.10.135
# 默认27017
port = 27003 
#注意:不需要显式的去指定主从,主从是动态选举的 
#副本集集群,需要指定一个名称,在一个副本集下,名称是相同的 
replSet=rs001
```

## 节点2配置

```shell
# 数据库文件位置 
dbpath=/root/mongodb/bin/rs/slave/data
#日志文件位置 
logpath=/root/mongodb/bin/rs/slave/logs/mongodb.log # 以追加方式写入日志
logappend=true
# 是否以守护进程方式运行
fork = true
bind_ip=192.168.10.135
# 默认27017
port = 27004 
#注意:不需要显式的去指定主从,主从是动态选举的 
#副本集集群,需要指定一个名称,在一个副本集下,名称是相同的 
replSet=rs001
```

## 节点3配置

```shell
# 数据库文件位置 
dbpath=/root/mongodb/bin/rs/arbiter/data
#日志文件位置 
logpath=/root/mongodb/bin/rs/arbiter/logs/mongodb.log 
# 以追加方式写入日志
logappend=true
# 是否以守护进程方式运行
fork = true
bind_ip=192.168.10.135
# 默认27017
port = 27005 
#注意:不需要显式的去指定主从,主从是动态选举的 
#副本集集群,需要指定一个名称,在一个副本集下,名称是相同的 
replSet=rs001
```

## 配置主备和仲裁

需要登录到mongodb的客户端**进行配置主备和仲裁角色**。

注意创建dbpath和logpath

```shell
mongo 192.168.10.135:27003
use admin
#配置
cfg={_id:"rs001",members: [
{_id:0,host:"192.168.10.135:27003",priority:2},
{_id:1,host:"192.168.10.135:27004",priority:1},
{_id:2,host:"192.168.10.135:27005",arbiterOnly:true}
]}
rs.initiate(cfg);
```

说明:

* cfg中的_id的值是【副本集名称】
* priority:数字越大,优先级越高。优先级最高的会被选举为主库
* arbiterOnly:true,如果是仲裁节点,必须设置该参数

测试

```shell
rs.status()
```

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-06-082912.png)

## 无仲裁副本集

​	和有仲裁的副本集基本上完全一样,只是在admin数据库下去执行配置的时候,不需要指定优先级和仲裁节点。这种情况,如果节点挂掉,那么他们都会进行选举。

```shell
mongo 192.168.10.135:27006
use admin
cfg={_id:"rs002",members: [
{_id:0,host:"192.168.10.135:27006"},
{_id:1,host:"192.168.10.135:27007"},
{_id:2,host:"192.168.10.135:27008"}
]}
rs.initiate(cfg);
```