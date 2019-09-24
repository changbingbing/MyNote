# 3.1 将 zookeeper 源码导入到 Idea

​	由于 **Zookeeper** 是使用 **Ant** 进行构建的,所以若想将 **Zookeeper** 的源码导入到 **Eclipse** 或**Idea** 中进行源码分析,需要首先安装 **Ant** 工具,然后再执行 `ant eclipse` 命令,会生成 **Eclipse**工程,然后再导入到 **Eclipse** 或 **Idea** 中。

## 3.1.1 下载并安装Ant

### （1）下载Ant

​	[官网下载](http://ant.apache.org/)

### （2）安装配置Ant

​	将下载的 **Ant** 安装 **zip** 包解压到任意目录,例如 `D:\course` 中。然后在系统环境变量中添加 `ANT_HOME`,并将 **Ant** 安装目录下的 **bin** 目录添加到系统环境变量 **PATH** 中。

![](https://ws2.sinaimg.cn/large/006tKfTcly1g0fgryk1v4j30jj0bpdhr.jpg)

![](https://ws2.sinaimg.cn/large/006tKfTcly1g0fgscrin7j30tj0vhn5l.jpg)

​	在命令行窗口的任意目录下执行 `Ant –version` 命令,可以看到版本号,则说明 **Ant** 安装成功。

![](https://ws1.sinaimg.cn/large/006tKfTcly1g0fgt3te6sj30hf05waav.jpg)

## 3.1.2 构建Eclipse工程

​	在命令行窗口中进入到 **zk** 解压目录,执行 `ant eclipse` 命令。其会根据 `build.xml` 文件对当前的 **zookeeper** 源码进行构建,将其构建为一个 **Eclipse** 工程。不过,对于该构建过程,其**JDK** 最好能够满足 `build.xml` 文件的要求。打开 `build.xml` 文件,可以看到 **zk** 源码是使用 **JDK6**编写并编译的,所以最好使用该指定的 **JDK**,这样可以保证将来构建出的 **Eclipse** 工程中没有错误。

![](https://ws4.sinaimg.cn/large/006tKfTcly1g0fgv02ofbj30v90b244d.jpg)

​	不过,由于我们这里仅仅跟踪 **Leader** 的选举算法,本机使用的是 **JDK10**,构建出的该**Eclipse** 工程中的相关代码并没有报错,所以这里就不更换 **JDK** 版本了

![](https://ws4.sinaimg.cn/large/006tKfTcly1g0fgymuuf0j30jz06dab6.jpg)

![](https://ws4.sinaimg.cn/large/006tKfTcly1g0fgw2hqryj30vu0icjw3.jpg)

构建成功后,在 **zk** 源码目录中可以看到 **Eclipse** 工程的相关文件。

![](https://ws3.sinaimg.cn/large/006tKfTcly1g0fgyaslvjj30fe0nw77j.jpg)

## 3.1.3 导入Idea

打开 Idea,选择导入工程,找到 zk 的源码解压目录,直接导入。

![](https://ws1.sinaimg.cn/large/006tKfTcly1g0fgxnmc2bj30f90hvwgj.jpg)

![](https://ws1.sinaimg.cn/large/006tKfTcly1g0fgz45gzuj30ij0bndhs.jpg)

![](https://ws2.sinaimg.cn/large/006tKfTcly1g0fgzhpj9bj30ld0cptcg.jpg)



# 3.2 选举算法源码中的总思路

​	**Zookeeper** 支持三种选举算法,但默认的是 **`FastLeaderElection`** 算法,该算法是 **ZAB 协议**在 **Leader** 选举中的工程应用,所以直接找到该类对其进行分析。该类中的最为重要的方法为 `lookForLeader()`,是选举 **Leader** 的核心方法。该方法大体思路可以划分为三块:

## 3.2.1 选举前的准备工作

​	选举前需要做一些准备工作,例如,**创建选举对象**、**创建选举过程中需要用到的集合**、**初始化选举时限**等。 

## 3.2.2 将自己作为初始化Leader投出去

​	在当前 **Server** 第一次投票时会先将自己作为 **Leader**,然后将自己的选票广播给其它所有 **Server**。


## 3.2.3 验证自己的投票与大家的投票谁更适合做 Leader

​	在“我选我”后,当前 **Server** 同样会接收到其它 **Server** 发送来的选票通知(`Notification`)。 通过 while 循环,遍历所有接收到的选票通知,比较谁更适合做 **Leader**。若找到一个比自己更适合的 **Leader**,则修改自己选票,重新将新的选票广播出去。 

​	当然,在此过程中还要查看自己接收到的选票是否是所有其它 Server 发送的选票。若不够,则说明自己的选票其它 **Server** 也可能没有接收到,则重新发送自己的选票。 

​	最终,在对某 **Server** 的选票超出半数时,新的 **Leader** 产生。然后再做一些收尾工作, 例如**清空选举过程中所使用的集合,以备下次使用**;再例如,**生成最终的选票,以备其它 Server 来同步数据**。 

# 3.3 源码解读

​	需要注意,对源码的阅读主要包含两方面。一个是对重要**类**、重要**成员变量**、重要**方法**的注释的阅读;一个是对重要方法的业务逻辑的分析。