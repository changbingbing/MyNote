# 1.1 系统架构的发展

## 1.1.1 单体系统架构

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0unkrwjz2j30n40h8gof.jpg)

​	当站点功能与流量都很小时,只需一个应用,将所有功能都集中在一个工程中,并部署在一台服务器上,以减少部署节点和成本。例如,将用户模块、订单模块、支付模块等都做在一个工程中,以一个应用的形式部署在一台服务器上。

* 优点：部署简单、成本低

* 缺点：就一个服务器，存在单点问题

## 1.1.2 集群架构

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0untegeskj30nc0k878c.jpg)

​	当流量增加而单体架构已无法应用其访问量时,可通过搭**建集群增加主机**的方式提升系统的性能。这种方式称为==水平扩展==。

## 1.1.3 分布式架构

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0unuv0p9qj30ui0qagrx.jpg)

​	当访问量逐渐增大,集群架构的水平扩展,其所带来的效率提升越来越小。此时可以**将项目拆分成多个功能相对独立的子工程**以提升效率。例如用户工程、订单工程、支付工程等。这种称为==垂直扩展==。

* 优点：可以根据工程功能、访问量的不同来配置不同数量机器

## 1.1.4 微服务架构

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0unyfoie1j311a0nq7b5.jpg)

​	当子工程越来越多时,发现它们可能同时都拥有某功能相同或相似的模块（代码冗余导致维护困难）,于是将这些在整个项目中冗余的功能模块抽取出来作为独立的工程（微服务）,这些工程就是专门为那些调用它们的工程服务的。那么这些抽取出的功能就称为微服务,微服务应用称为**服务提供者**,而调用微服务的应用就称为**服务消费者**。

## 1.1.5 流动计算架构

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0vnre9kjnj31380n2tfz.jpg)

​	随着功能的扩张,微服务就需要越来越多;随着 PV 的增涨,消费者工程就需要越来越多;随着消费者的扩张,为其提供服务的提供者服务器就需要越来越多,且每种提供者都要求创建为集群。这样的话,消费者对于提供者的访问就不能再采用直连方式进行了,此时就需要**服务注册中心**了。提供者将服务注册到注册中心,而消费者通过注册中心进行消费,消费者无需再与提供者绑定了。提供者的宕机,对消费者不会产生直接的影响。 

​	随着服务的增多,在一些特殊时段(例如双 11)就会出现服务资源浪费的问题:有些服务的 QPS 很低,但其还占用着很多系统资源,而有些 QPS 很高,已经出现了资源紧张, 用户体验骤降的情况。此时就需要服务治理中心了。让一些不重要的服务暂时性降级,或为其分配较低的权重等,对整个系统资源进行统一调配。 

​	这里的资源调配分为两种:**预调配**与**实时调配**：

* 预调配是根据系统架构师的“==系统容量预估==”所进行的调配,是一种经验,是一种预处理,就像每年双 11 期间的 PV 与 UV 都会很高,就需要提前对各服务性能进行调配。 
* 实时调配指的是根据服务监控中心所提供的基于访问压力的实时系统容量评估数据,对各服务性能进行实时调配的方案。 

# 1.2 小插曲－架构师的基本素养

## 1.2.1 基本概念

### （1）系统容量与系统容量预估

​	系统容量指系统所能承受的**最大访问量**,而系统容量预估则是**在峰值流量到达之前系统架构师所给出的若干技术指标值**。常用的技术指标值有:`QPS`、`PV`、`UV`、`并发量`、`带宽`、`CPU使用率`、`内存硬盘占用率`等。系统容量预估是架构师必备的技能之一。

### （2）QPS

​	**`QPS`**：**Query Per Second**,每秒查询量。在分布式系统中 QPS 的定义是：**单个进程每秒请求服务器的成功次数**。QPS 一般可以通过压力测试工具测得,例如 **LoadRunner**、**Apache JMeter**、**NeoLoad**、**http_load** 等。

```xml
QPS=总请求数/进程总数/请求时间=总请求数/(进程总数* 请求时间)
```

### （3）UV

​	**Unique Visitor**：**独立访客数量,指一定时间范围内站点访问所来自的 IP 数量**。同一 IP多次访问站点只计算一次。一般以 24 小时计算。 

### （4）PV

**Page View**：**页面访问量,指一定时间范围内打开或刷新页面的次数**。一般以 `24 小时`计算。

## 1.2.2 系统容量预估基本计算

### （1）带宽计算

平均带宽的计算公式为:

```shell
平均带宽 = 总流量数(bit) / 产生这些流量的时长(秒) 
		=(PV * 页面平均大小 * 8) / 统计时间(秒)
#说明:公式中的 8 指的是将 Byte 转换为 bit,即 8b/B,因为带宽的单位是 bps(比特率),即 bit per second,每秒二进制位数,而容量单位一般使用 Byte。
```

​	假设某站点的日均 PV 是 10w,页面平均大小 0.4 M,那么其平均带宽需求是:平均带宽 = (10w*0.4M*8) /(60*60*24)=3.7Mbps

​	以上计算的仅仅是平均带宽,我们在进行容量预估时需要的是峰值带宽,即必须要保证站点在峰值流量时能够正常运转。假设,峰值流量是平均流量的 5 倍,这个 5 倍称为峰值因子。按照这个计算,实际需要的带宽大约在 3.7 Mbps * 5=18.5 Mbps 。 

```shell
带宽需求 = 平均带宽 * 峰值因子 
```

### （2）并发量计算

​	并发量,也称为并发连接数,一般是指单台服务器在一个响应周期内的连接数。平均并发连接数的计算公式是:

```shell
平均并发连接数 =(站点PV* 页面平均衍生连接数)/(统计时间 * 响应周期 * web 服务器数量);
#注:页面平均衍生连接数是指,一个页面请求所产生的 http 连接数量,如对静态资源 的 css、js、images 等的请求数量。这个值需要根据实际情况而定。
```

​	例如,一个由 5 台 web 主机构成的集群,其日均 PV 50w,每个页面平均 30 个衍生连接, http 响应周期为 0.5 秒,则其平均并发连接数为: 平均并发量 = (50w*30)/(60*60*24*0.5*5)=69 

​	若峰值因子为 6,则峰值并发量为: 峰值并发量 = 平均并发量 * 峰值因子 = 69 * 6 = 414 

### （3）服务器预估量

​	根据往年同期活动获得的日均 PV、并发量、页面衍生连接数,及公司业务扩展所带来的流量增涨率,就可以计算出服务器预估值。

```shell
服务器预估值 =(PV * 页面衍生连接数*(1 + 增涨率)) / (统计时间 * 响应周期) / 单机并发连接数
```

# 1.3 分布式技术图谱

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0xirkyqasj31810u013g.jpg)

# 1.4 Dubbo简介

## 1.4.1 官网简介

​	Dubbo 官网为 http://dubbo.apache.org。该官网是 Dubbo 正式进入 Apache 开源孵化器后改的。Dubbo 原官网为:http://dubbo.io 。 

​	Dubbo 官网已做过了中英文国际化,用户可在中英文间任何切换。

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0xk0bwullj30te0c6wgf.jpg) 

## 1.4.2 什么事PRC

> ​	RPC(Remote Procedure Call Protocol)——远程过程调用协议,它是一种<u>通过网络</u>从远程计算机程序上请求服务,而不需要了解底层网络技术的协议。RPC 协议假定某些传输协议的存在,如 TCP 或 UDP,为通信程序之间携带信息数据。在 OSI 网络通信模型(OSI 七层网络模型,OSI,Open System Interconnection,开放系统互联)中,RPC 跨越了传输层和应用层。RPC 使得开发包括网络分布式多程序在内的应用程序更加容易。 
>
> ​	RPC **采用客户机/服务器模式**(即 C/S 模式)。请求程序就是一个客户机,而服务提供程序就是一个服务器。首先,客户机调用进程发送一个有进程参数的调用信息到服务进程,然 后等待应答信息。在服务器端,进程保持睡眠状态直到调用信息到达为止。当一个调用信息到达,服务器获得进程参数,计算结果,发送答复信息,然后等待下一个调用信息,最后, 客户端调用进程接收答复信息,获得进程结果,然后调用执行继续进行。
>
> ​	PRC的底层实际上是Netty，Netty的底层是NIO ，NIO底层是Socket。
>
> ​	PRC通讯效率很高，实际上就是因为底层是Socket编程。也就是Socket通讯，Socket通讯需要主机名、端口号就可以通讯。

## 1.4.3 Dubbo重要时间点

Dubbo 发展过程中的重要时间点: 

> * 2011 年开源,之后就迅速成为了国内该类开源项目的佼佼者。2011 年时,优秀的、可在生产环境使用的 RPC 框架很少,Dubbo 的出现迅速给人眼前一亮的感觉,迅速受到了开发者的亲睐。 
> * 2012 年 10 月之后就基本停止了重要升级,改为阶段性维护。
> * 2014 年 10 月 30 日发布 2.4.11 版本后,Dubbo 停止更新。 
> * 2017 年 10 月云栖大会上,阿里宣布 Dubbo 被列入集团重点维护开源项目,这也就意味着 Dubbo 起死回生,开始重新进入快车道。 
> * 2018 年 2 月 15 日,大年三十,经过一系列紧张的投票,宣布 Dubbo 正式进入Apache 孵化器。

# 1.5 Dubbo四大组件

Dubbo 中存在四大组件: 

> *   Provider:服务提供者。 
> *   Consumer:服务消费者。
> *   Registry:服务注册与发现的中心,提供目录服务,亦称为服务注册中心
> *   Monitor:统计服务的调用次数、调用时间等信息的日志服务,并可以对服务设置权限、 降级处理等,称为服务管控中心

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0xkgha07xj30rw0kmq48.jpg)

# 1.6 版本号

## 1.6.1 Dubbo版本号与客户端

```xml
<dependency>
   <groupId>org.apache.dubbo</groupId>
   <artifactId>dubbo</artifactId>
   <version>2.7.0</version>
</dependency>
```

注意：2.6.0之前版本都属于进入孵化器之前的版本，2.6.1之后版本是进入孵化器后的版本，这是一个分水岭。

> [dubbo-2.6.1](https://github.com/apache/incubator-dubbo/releases/tag/dubbo-2.6.1)
>
> [![@chickenlj](https://avatars3.githubusercontent.com/u/18097545?s=40&v=4)](https://github.com/chickenlj) [chickenlj](https://github.com/chickenlj) released this on 2 Apr 2018 · [1040 commits](https://github.com/apache/incubator-dubbo/compare/dubbo-2.6.1...master) to master since this release
>
> ## Refactor
>
> * Stripping OPS modules such as dubbo-admin, dubbo-monitor, etc. to a separate repo, [#1209](https://github.com/apache/incubator-dubbo/issues/1209).
> * Refactor project dependency and package structure: serialization as a standalone module, all submodules can deploy to remote repository separately, [#1322](https://github.com/apache/incubator-dubbo/issues/1322).
>
> ## Enhancements & Bugfixes
>
> * Support build in java 9, [#1283](https://github.com/apache/incubator-dubbo/issues/1283).
> * Avoid blocking problems of hessian serailization in high concurrency scenarios, [#1196](https://github.com/apache/incubator-dubbo/pull/1196).
> * **Change default ZK client to "curator"**, [#1186](https://github.com/apache/incubator-dubbo/issues/1186).
> * Avoid retry when parameter validation fails, [#1031](https://github.com/apache/incubator-dubbo/pull/1031).
> * Remove dubbo destroy logic from spring bean destroy process.
> * Add "oninvoke" parse, [#1353](https://github.com/apache/incubator-dubbo/pull/1353).
> * Other small bugs or polish.
>
> ## 重构
>
> * **将dubbo-admin, dubbo-monitor等OPS模块剥离到一个单独的工程仓库中**, [#1209](https://github.com/apache/incubator-dubbo/issues/1209).
> * 重构工程依赖和包结构: 序列化作为一个单独的模块，所有的子模块可以单独部署到远程仓库中, [#1322](https://github.com/apache/incubator-dubbo/issues/1322).

## 1.6.2 Dubbo与Spring的版本号

Dubbo依赖的Spring版本是4.3.16，因此我们项目使用的spring版本最好与此保持一致！

```xml
<spring-version>4.3.16.RELEASE</spring-version>
```

