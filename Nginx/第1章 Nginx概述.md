# 1.1 Nginx简介

​	Nginx (engine x) 是一个**轻量级**、**高性能**、**基于 Http** 的**反向代理服务器**、**静态 web 服务器**。 

​	Nginx 最初是由俄罗斯人 Igor Sysoev(伊戈尔·赛索耶夫)使用 C 语言为俄罗斯访问量第二的 Rambler.ru 站点开发的一款服务器。2004 年 10 月发布第一个版本。 

​	Nginx 的官网: http://nginx.org 

​	国内大型的站点,例如百度、京东、新浪、网易、腾讯、淘宝等,都使用了 Nginx。

​	 https://www.netcraft.com/ 

# 1.2 代理服务器

## 1.2.1 正向代理

​	客户端知道访问对象是谁，机器架设在客户端！

### （1）隐藏

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1ai0w6q0qj30ve0h2mzs.jpg)

* 保护和隐藏客户端 

### （2）翻墙

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1aigrh8x9j30uy0k8tco.jpg)

### （3）提速

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1aihmnrixj30v20lkn0v.jpg)

### （4）缓存

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1aii69q5lj30ua0isn18.jpg)

* 典型用类： CDN、Maven

### （5）授权

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1aijcv3e6j30zs0kogqg.jpg)

## 1.2.2 反向代理

​	保护隐藏服务器，机器架设在服务端！

### （1）保护隐私

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1aikz9rtjj30xw0iajun.jpg)

### （2）分布式路由

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1aimnglejj30ws0k2gq1.jpg)

### （3）负载均衡

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1aipyuxzbj30wg0jun1m.jpg)

### （4）动静分离

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1air0ywnjj30w40rowkj.jpg)

### （5）数据缓存

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1airzlw0fj30vg0iy42v.jpg)

## 1.2.3 正向代理和反向代理区别

* 客户端是否知道自己要访问的服务器是谁；
* 机器架设的位置不同；

# 1.3 Nginx的特点

## 1.3.1 高并发

​	一个 Nginx 服务器在不做任何配置的情况下**并发量可达 1000 左右**。在硬件条件允许的前提下,Nginx 可以支持高达 **5-10 万**的并发量(除了 Nginx 的设置外,Linux 主机需要做大量的设置来配合 Nginx)。 

​	对比一下 Tomcat。Tomcat 服务器默认的并发量为 **150**(不做任何配置)。即,当有超过 150 个用户同时访问某 Servlet 时,Tomcat 的响应就会变得非常慢。 

## 1.3.2 低消耗

​	官方给出的测试结果,10000 个非活跃连接,在 Nginx 中仅消耗 2.5M 内存。对于一般性的 DoS 攻击来说就不是事儿,但对于 DDoS 也会是问题。

拓展：

> 网络安全的几个维度：保密性、可用性、安全性
>
> * [DOS](https://baike.baidu.com/item/dos攻击)：Denial of Service的简称，即**拒绝服务**！造成DoS的攻击行为被称为DoS攻击，其目的是使计算机或网络无法提供正常的服务。最常见的DoS攻击有**计算机网络带宽攻击**和**连通性攻击**。即：攻击其可用性，使服务拒绝服务！生活举例：在ATM机恶意取款，一张一张地取，ATM虽名义上在正常提供服务，但拒绝了其它人的正常取款；
> * [DDOS](https://baike.baidu.com/item/分布式拒绝服务攻击?fromtitle=DDOS&fromid=444572)：Distributed Denial of Service的简称，即**分布式拒绝服务**！攻击指借助于客户/服务器技术，将多个计算机联合起来作为攻击平台，对一个或多个目标发动DDoS攻击，从而成倍地提高拒绝服务攻击的威力。通常，攻击者使用一个偷窃帐号将DDoS主控程序安装在一个计算机上，在一个设定的时间主控程序将与大量代理程序[通讯](https://baike.baidu.com/item/%E9%80%9A%E8%AE%AF/396194)，代理程序已经被安装在网络上的许多计算机上。代理程序收到指令时就发动攻击。利用客户/[服务](https://baike.baidu.com/item/%E6%9C%8D%E5%8A%A1)器技术，主控程序能在几秒钟内激活成百上千次代理程序的运行！代理程序(肉鸡)好找，攻击成本低，防不胜防，预防成本高！生活举例：超市闹事

## 1.3.3 热部署

​	可以在 7*24 小时不间断服务的前提下,进行 Nginx 版本的平滑升级,Nginx 配置文件的平滑修改。即在不停机的情况下升级 Nginx（灰度升级）,修改替换 Nginx 配置文件。

​	能做到热部署，主要还是因其**高可用**！

## 1.3.4 高可用

​	Nginx 之所以可以实现高并发,是因为其**具有很多工作进程 worker**。当这些工作进程中的某些出现问题停止工作时,并不会影响整个系统的整体运行。因为其它 worker 会接替那些出问题的线程。

## 1.3.5 高扩展

​	Nginx 之所以现在的用户很多,是因为很多功能都已经**开发好并模块化**。若需要哪些功能,只需要安装相应功能的扩展模块即可。根据编写扩展模块所使用的语言的不同,可以划分为两类：**C 语言扩展模块**与 **LUA 脚本扩展模块**。 

> [OpenResty](http://openresty.org/cn/)® 是一个基于 [Nginx](http://openresty.org/cn/nginx.html) 与 Lua 的高性能 Web 平台，其内部集成了大量精良的 **Lua 库**、**第三方模块**以及大多数的**依赖项**。用于方便地搭建能够处理超高并发、扩展性极高的**动态 Web 应用**、**Web 服务**和动态**网关**。

# 1.4 Nginx的下载与安装

## 1.4.1 Nginx的下载

​	官网下载：http://nginx.org/

## 1.4.2 Nginx的源码安装

### （1）安装Nginx

#### A、上传Nginx

​	将下载好的 Nginx 上传到新复制的主机的/usr/tools 目录。

```shell
scp -r changbingbing@192.168.36.173:/Users/changbingbing/Downloads/nginx-1.12.2* ./
```

#### B、安装gcc

​	由于 Nginx 是由 C/C++语言编写的,所以对其进行编译就必须要使用相关编译器。对于C/C++语言的编译器,使用最多的是 gcc 与 gcc-c++,而这两款编译器在 CentOS7 中是没有安装的,所以首先要安装这两款编译器。

```shell
yum -y install gcc gcc-c++
```

#### C、安装依赖库

​	基本的 Nginx 功能依赖于一些基本的库,在安装 Nginx 之前需要提前安装这些库。

```shell
yum -y install pcre-devel openssl-devel
```

#### D、创建解压目录

​	在/usr 下创建 apps 目录,用于存放解压后的安装包程序。

```shell
mkdir /usr/apps
```

#### E、解压Nginx

```shell
tar -zxvf nginx-1.12.2.tar.gz -C /usr/apps/
```

#### F、生成 makefile

​	在 Nginx 解压目录下运行 make 命令,用于完成编译。但此时会给出提示:没有指定目标,并且没有发现编译文件 makefile 。

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1bdiym4owj30wk05q0vd.jpg)

​	编译命令 make 需要根据编译文件 makefile 进行编译,所以在编译之前需要先生成编译
文件 makefile。使用 configure 命令可以生成该文件。

```shell
./configure
```

#### G、编译安装

```shell
make && make install
```

### （2）使nginx命令随处可用

​	在 Nginx 的安装目录/usr/local/nginx 中有一个 sbin 目录,其中存放着 nginx 的命令程序nginx。

```shell
#nginx 的命令程序nginx
ll /usr/local/nginx/sbin/

#创建软连接
ln -s /usr/local/nginx/sbin/nginx /usr/local/sbin/

#查看软连接
ll /usr/local/sbin/
```

## 1.4.3 Nginx命令

### （1）查看命令选项nginx -h

```shell
#查看 Nginx 命令的选项
nginx -h
```

### （2）查看Nginx版本信息nginx -v或-V

```shell
#显示 Nginx 版本信息
nginx -v

#显示更多的版本相关信息,例如 gcc 的版本,OpenSSL 的版本等。
nginx -V
```

### （3）启动命令nginx -c file

```shell
#nginx –c(小写字母)可启动 Nginx,启动成功后无任何提示。
nginx -c /usr/local/nginx/conf/nginx.conf

#若不指定配置文件,则默认加载的是 Nginx 安装目录下的 conf/nginx.cnf 
nginx
```

### （4）测试配置文件命令nginx -tq

```shell
#测试配置文件是否正确,默认只测试默认的配置文件 conf/nginx.conf
nginx -t

#测试配置文件是否正确,并显示配置文件内容。
nginx -T

#在配置文件测试过程中,禁止显示非错误信息,即只显示错误信息。
nginx -tq

#可以结合-c 选项指定要测试的配置文件。注意,其不会启动 nginx。
nginx -c /usr/local/nginx/conf/nginx.conf -t
```

### （5）停止命令nginx -s stop/quit

```shell
#在 nginx 命令后通过-s 选项,可以指定不同的信号完成不同的功能

#强制停止 Nginx,无论当前工作进程是否正在处理工作
nginx –s stop

#优雅停止 Nginx,使当前的工作进程完成当前工作后停止。
nginx –s quit
```

### （6）平滑重启命令nginx -s reload

```shell
#在不重启 Nginx 的前提下重新加载 Nginx 配置文件,称为平滑重启。
nginx -s reload
```

### （7）nginx -s reopen  ?

```shell
#重新打开日志文件。
nginx -s reopen
```

### （8）nginx -p

```shell
#指定 Nginx 配置文件的存放路径。
#-p prefix : set prefix path (default: /usr/local/nginx/)
nginx -p
```

### （9）nginx -g

```shell
#设置配置文件以外的全局指令
#-g directives : set global directives out of configuration file
nginx -g
```

## 1.4.4 页面访问测试

### （1）关闭防火墙

```shell
service iptables stop

chkconfig iptables off
```

### （2）浏览器访问

​	由于 Nginx 服务器默认的端口号为 80,所以在浏览器中直接输入 Nginx 的主机名或 IP,就可以看到 Nginx 欢迎页面。只要可以看到以下页面信息,则说明 Nginx 安装运行成功。

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1bln90vrjj30nf0a8jsm.jpg)
