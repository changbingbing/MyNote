# 2.1 Nginx 性能调优

​	在 Nginx 性能调优中,有**两个非常重要的理论点(面试点)**需要掌握。所以,下面首先讲解这两个知识点,再进行**性能调优**的配置。

## 2.1.1  零拷贝（Zero Copy）

### （1）零拷贝基础

​	零拷贝指的是：**从一个存储区域到另一个存储区域的 copy 任务没有 CPU（CPU的IOU） 参与**。取决于操作系统是否配置了支持，与java本身没有关系。零拷贝通常用于网络文件传输,以减少 ==CPU 消耗==和==内存带宽占用==,减少==用户空间==与 ==CPU 内核空间==的拷贝过程,减少==用户上下文==与==CPU 内核上下文==间的切换,提高系统效率。（这些切换相当的消耗资源）

> 用户空间：指用户可操作的内存缓存区域;
>
> CPU 内核空间：指仅 CPU 可以操作的寄存器、缓存及内存缓存区域；
>
> 用户上下文：指用户状态环境、参数；
>
> CPU 内核上下文：指 CPU 内核状态环境。 

​	零拷贝需要 DMA 控制器的协助。DMA（Direct Memory Access）,直接内存存取,是 CPU 的组成部分,其可以在 CPU 内核(算术逻辑运算器 ALU 等)不参与运算的情况下将数据从一个地址空间拷贝到另一个地址空间。

### （2）传统拷贝方式

​	下面均以“将一个硬盘中的文件通过网络发送出去”的过程为例,来详细分析不同拷贝方式的实现细节。 

#### A、实现细节

​	首先通过应用程序的 `read()`方法将文件从硬盘读取出来,然后再调用 `send()`方法将文件发送出去。

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1bmifxrdgj30zg0q2qb9.jpg) 

#### B、总结

​	该拷贝方式共进行了 **4 次用户空间与内核空间的上下文切换**,以及 **4 次数据拷贝**,其中两次拷贝存在 CPU 参与。 

​	我们发现一个很明显的问题：应用程序的作用仅仅就是一个数据传输的中介,最后将 `kernel buffer` 中的数据传递到了 `socket buffer`。显然这是没有必要的。所以就引入了**零拷贝**。 

### （3）零拷贝方式

#### A、实现细节

​	Linux 系统(CentOS6 及其以上版本)对于零拷贝是通过 **sendfile 系统**调用实现的。

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1bncr7i04j30xc0own3o.jpg)

#### B、总结

​	该拷贝方式共进行了 **2 次用户空间与内核空间的上下文切换**,以及 **3 次数据拷贝**,但整个拷贝过程均没有 CPU 的参与,这就是**零拷贝**。 

​	我们发现这里还存在一个问题：`kernel buffer` 到 `socket buffer` 的拷贝需要吗?`kernel buffer` 与 `socket buffer` 有什么区别呢?DMA 控制器所控制的拷贝过程有一个要求,数据在源头的存放地址空间必须是**连续的**。**`kernel buffer` 中的数据无法保证其连续性**,所以需要将数据再拷贝到 `socket buffer`,**`socket buffer` 可以保证了数据的连续性**。 

​	这个拷贝过程能否避免呢?可以,只要主机的 DMA 支持 **`Gather Copy` 功能**,就可以避免由 `kernel buffer` 到 `socket buffer` 的拷贝。 

### （4）Gather Copy DMA 零拷贝方式

​	由于该拷贝方式是由 DMA 完成,与系统无关,所以**只要保证系统支持 sendfile 系统调用功能即可**。

#### A、实现细节

​	该方式中没有数据拷贝到 `socket buffer`。取而代之的是只是将 `kernel buffer` 中的数据**描述信息**写到了 `socket buffer` 中。数据描述信息包含了两方面的信息：`kernel buffer` 中数据的**地址**及**偏移量**。

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1bnlpcdbqj30va0nqwkq.jpg)

#### B、总结

​	该拷贝方式共进行了 **2 次用户空间与内核空间的上下文切换**,以及 **2 次数据拷贝**,并且==整个拷贝过程均没有 CPU 的参与==。 

​	该拷贝方式的系统效率是高了,但与传统相比,**也存在有不足**。传统拷贝中 `user buffer` 中存有数据,因此应用程序能够对数据进行修改等操作;零拷贝中的 user buffer 中没有了数据,所以应用程序**无法对数据进行操作**了。这时Linux 的 **mmap 零拷贝**解决了这个问题。 

### （5）mmap零拷贝

​	mmap 零拷贝是对零拷贝的改进。当然,若当前主机的 DMA 支持 `Gather Copy`,mmap同样可以实现 `Gather Copy DMA` 的零拷贝。 

#### A、实现细节

​	该方式与零拷贝的唯一区别是：**应用程序与内核共享了 `Kernel buffer`**。由于是共享,所以应用程序也就可以操作该 buffer 了。当然,应用程序对于 Kernel buffer 的操作,就会**引发用户空间与内核空间的相互切换**。

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1bnrcb0foj30xm0oy45q.jpg)

#### B、总结

​	该拷贝方式共进行了 **4 次用户空间与内核空间的上下文切换**,以及 **2 次数据拷贝**,并且==整个拷贝过程均没有 CPU 的参与==。虽然较之前面的零拷贝增加了两次上下文切换,但应用程序可以对数据进行修改了。

## 2.1.2 多路复用器 select|poll|epoll

### （1）基础知识

​	若要理解 `select`、`poll` 与 `epoll` 多路复用器的工作原理,就需要首先了解什么是多路复用器。而要了解什么是多路复用器,就需要先了解什么是“多进程/多线程连接处理模型”。

> 进程状态：就绪、阻塞、运行 

#### A、多线程／多进程连接处理模型

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1bnyo3bpij30ww0fejw8.jpg)

​	在该模型下,一个用户连接请求会由一个内核进程处理,而一个内核进程会创建一个应用程序进程,即 app 进程来处理该连接请求。应用程序进程在调用 IO 时,采用的是 **BIO 通讯方式**,即<u>应用程序进程在未获取到 IO 响应之前是处于**阻塞态**的。</u> 

​	该模型的优点是：内核进程不存在对 app 进程的竞争,一个内核进程对应一个 app 进程。 但也正因为如此,所以其弊端也就显而易见了。需要创建过多的 app 进程,而该创建过程十分的消耗系统资源。且一个系统的进程数量是有上限的,所以**该模型不能处理高并发的情况**。 

#### B、多路复用连接处理模型

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1bo2wi0v1j30tw0dc41z.jpg)

​	在该模型下,只有一个 app 进程来处理内核进程事务,且 app 进程一次只能处理一个内核进程事务。故这种模型对于内核进程来说,**存在对 app 进程的竞争**。 

​	在前面的“多进程/多线程连接处理模型”中我们说过,app 进程只要被创建了就会执行内核进程事务。那么在该模型下,应用程序进程应该执行哪个内核进程事务呢?谁的准备就绪了,app 进程就执行哪个。但 app 进程怎么知道哪个内核进程就绪了呢?需要通过“**多路复用器**”来获取各个内核进程的状态信息。那么多路复用器又是怎么获取到内核进程的状态信息的呢?不同的多路复用器,其算法不同。常见的有三种:`select`、`poll` 与 `epoll`。
​	app 进程在进行 IO 时,其采用的是 **NIO 通讯方式**,即<u>该 app 进程不会阻塞。当一个 IO结果返回时,app 进程会暂停当前事务,将 IO 结果返回给对应的内核进程。然后再继续执行暂停的线程。</u> 

​	该模型的优点很明显,无需再创建很多的应用程序进程去处理内核进程事务了,仅需一个即可。 

### （2）多路复用器工作原理

#### A、select

​	`select` 多路复用器是**采用轮询的方式**,一直在轮询所有的相关内核进程,查看它们的进程状态。若已经就绪,则马上将该内核进程放入到**就绪队列**。否则,继续查看下一个内核进程状态。在处理内核进程事务之前,app 进程首先会从内核空间中将用户连接请求相关数据复制到用户空间。 

该多路复用器的**缺陷**有以下几点:

* 对所有内核进程**采用轮询方式效率会很低**。因为对于大多数情况下,内核进程都不属于就绪状态,只有少部分才会是就绪态。所以这种轮询结果大多数都是无意义的。 
* 由于**就绪队列底层由数组实现**,所以其所能处理的内核进程数量是有限制的,即其**能够处理的最大并发连接数量是有限制**的。
* 从内核空间到用户空间的复制,**系统开销大**。 

#### B、poll

​	`poll` 多路复用器的工作原理与 select 几乎相同,不同的是,由于其**就绪队列由链表实现**, 所以,其对于要处理的内核进程数量理论上是没有限制的,即其**能够处理的最大并发连接数量是没有限制的**(当然,要受限于当前系统中进程可以打开的最大文件描述符数 ulimit,后面会讲到)。 

#### C、epoll

​	`epoll` 多路复用是对 `select` 与 `poll` 的增强与改进。其不再采用轮询方式了,而是**采用==回调==方式**实现对内核进程状态的获取，一旦内核进程就绪,其就会回调 `epoll` 多路复用器,进入到多路复用器的就绪队列(**由链表实现**)。所以 `epoll` 多路复用模型也称为 `epoll` ==事件驱动模型==。 

​	另外,应用程序所使用的数据,也不再从内核空间复制到用户空间了,而是**使用 mmap 零拷贝**机制,大大降低了系统开销。 

​	当内核进程就绪信息通知了 epoll 多路复用器后,多路复用器就会马上对其进行处理, 将其马上存放到就绪队列吗?不是的。根据处理方式的不同,可以分为两种处理模式:**`LT` 模式**与 **`ET` 模式**。 

##### a、LT模式

​	LT（Level Triggered）**水平触发模式**。即只要内核进程的就绪通知由于某种原因暂时没有被 epoll 处理,则该内核进程就会定时将其就绪信息通知 `epoll`。直到 `epoll` 将其写入到就绪队列,或由于某种原因该内核进程又不再就绪而不再通知。其支持两种通讯方式:`BIO` 与`NIO`。

##### b、ET模式

​	ET（Edge Triggered）**边缘触发模式**。其仅支持 `NIO` 的通讯方式。 

​	当内核进程的就绪信息**仅会通知一次 `epoll`**,无论 `epoll` 是否处理该通知。明显该方式的效率要高于 LT 模式,但其有可能会出现就绪通知被忽视的情况,即**连接请求丢失**的情况。

#### D、总结

> *  select：采用轮询方式。就绪队列是数组，但数组有个问题就是大小固定，而且实际场景中，大部分内核进程都处于非就绪状态，但还得不停的轮询，这就很消耗无谓的资源性能了；
> * poll：采用轮询方式。就绪队列是链表，链表相对数组操作效率高了些；
> * epoll：采用回调方式，谁就绪谁调多路复用器，效率高，资源节约；
>   * LT模式：水平触发模式，底层通讯方式有BIO（BIO默认）与NIO(NIO需要设置)，会发送多次就绪通知；
>   * ET模式：边缘触发模式，就绪通知就发一次，爱处理不处理，效率高一些，但存在丢失问题！底层通讯方式NIO

​	链接活跃时，轮询的效率高一些；链接非活跃时，回调的效率高一些。所以说epoll未必比poll效率就高，是有前提的！

## 2.1.3 Nginx的并发处理机制

​	一般情况下并发处理机制有三种：**多进程**、**多线程**与**异步机制**。Nginx 对于并发的处理同时采用了三种机制。当然,其异步机制使用的是**异步非阻塞方式**。 

​	我们知道 Nginx 的进程分为两类：**master 进程**与 **worker 进程**。每个 master 进程可以生成多个 worker 进程,所以其是多进程的。每个 worker 进程可以同时处理多个用户请求,每个用户请求会由一个线程来处理,所以其是多线程的。 

​	那么,如何解释其“异步非阻塞”并发处理机制呢?

​	worker 进程采用的就是 **`epoll` 多路复用机制**来对后端服务器进行处理的。当后端服务器返回结果后,后端服务器就会回调 epoll 多路复用器,由多路复用器对相应的 worker 进程进行通知。此时,worker 进程就会挂起当前正在处理的事务,拿 IO 返回结果去响应客户端请求。响应完毕后,会再继续执行挂起的事务。这个过程就是“异步非阻塞”的。 

> 小结：
>
> 1. 异步非阻塞：异步是目的，非阻塞时手段；
> 2. Nginx并发处理机制：多进程、多线程、异步非阻塞方式；
> 3. maser进程主要负责worker进程的生命周期的，maser进程就一个，worker进程会有多个；一个worker进程会有多个worker线程。

## 2.1.4 全局模块下的调优

### （1）worker_processes 2

打开 /usr/apps/nginx-1.12.2/conf/nginx.conf 配置文件,可以看到 worker_processes 的默认值为 1。

```shell
#user  nobody;
worker_processes  1;
#......
```

* `worker_processes`,工作进程,用于指定 Nginx 的工作进程数量。其数值一般设置为 **CPU 内核数量**,或**内核数量的整数倍**。 

* 不过需要注意：该值不仅仅取决于 **CPU 内核数量**,还与**硬盘数量**及**负载均衡模式**相关。 在不确定时可以指定其值为 `auto`。 

  ```shell
  #user  nobody;
  worker_processes  auto;
  #......
  ```

### （2）worker_cpu_affinity 01 01

​	将 worker 进程与具体的内核进行绑定。不过,若指定 worker_processes 的值为 auto, 则无法设置 `worker_cpu_affinity`。 

```shell
#user  nobody;
worker_processes 2;
worker_cpu_affinity 01 10;
#......
```

​	该设置是通过二进制进行的。每个内核使用一个二进制位表示,0 代表内核关闭,1 代表内核开启。也就是说,有几个内核,就需要使用几个二进制位。

### （3）worker_rlimit_nofile 65535

​	用于设置一个 worker 进程所能打开的最多文件数量。其默认值与当前 Linux 系统可以打开的最大文件描述符数量相同。

```shell
#user  nobody;
worker_processes 2;
worker_cpu_affinity 01 10;
worker_rlimit_nofile 65535
#......
```

扩展：

```shell
#查看linux能打开最大文件数量值；
ulimit -n

#临时修改linux能打开最大文件数量值；
ulimit -n 56666

#全局设置：/etc/sysconfig
soft noifle 65535
hard noifle 65535
```

## 2.1.5 events模块下的调优

打开 /usr/apps/nginx-1.12.2/conf/nginx.conf 配置文件,即可看到events模块

### （1）worker_connections 1024

​	设置每一个 worker 进程可以并发处理的最大连接数。该值不能超过 `worker_rlimit_nofile` 

的值。 

### （2）accept_mutex on

* on:默认值,表示当一个新连接到达时,那些没有处于工作状态的 worker 将以**串行方式**来处理; ——通过互斥锁原理实现串行化。

* off:表示当一个新连接到达时,所有的 worker 都**会被唤醒**（由阻塞态变为就绪态）,不过只有一个 worker 能获取新连接,其它的 worker 会重新进入阻塞状态,这就是“==惊群==”现象。——对于nginx而言， “==惊群==”现象对其影响不是很大。

>小结：
>
>一般情况，高并发时设置为off，效率高！并发不高时，设置on好一些；

### （3）accept_nutex_delay 500ms

​	设置队首 worker 会尝试获取互斥锁的时间间隔。默认值为 500 毫秒。

### （4）multi_accept on

* off:系统会逐个拿出新连接按照负载均衡策略,将其分配给当前处理连接个数最少的 worker。 
* on:系统会实时的统计出各个 worker 当前正在处理的连接个数,然后会按照“缺编” 最多的 worker 的“缺编”数量,一次性将这么多的新连接分配给该 worker。 

### （5）use epoll

​	设置 worker 与客户端连接的处理方式。Nginx 会自动选择适合当前系统的最高效的方式。当然,也可以使用 use 指令明确指定所要使用的连接处理方式。user 的取值有以下几种:`select | poll | epoll | rtsig | kqueue | /dev/poll` 。

#### A、select｜poll｜epoll

​	这是三种多路复用机制。select 与 poll 工作原理几乎相同,而 epoll 的效率最高,是现在使用最多的一种多路复用机制。

#### B、rtsig（了解）

​	realtime signal,实时信号,Linux 2.2.19+的高效连接处理方式。但在 在 Linux 2.6 版本后,不再支持该方式。

#### C、kqueue（了解）

​	应用在 BSD 系统上的 epoll。

#### D、/dev/poll（了解）

​	UNIX 系统上使用的 poll。

## 2.1.6 http模块下的调优

### （1）非调优属性简介

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1f10qcvdsj30ru056q4p.jpg)

* `include mime.types;`
  将当前目录(conf 目录)中的 mime.types 文件包含进来。 
* `default_type application/octet-stream;`
  对于无扩展名的文件,默认其为 `application/octet-stream` 类型,即 Nginx 会将其作为一个八进制流文件来处理。 
* `charset utf-8;` 
  设置请求与响应的字符编码。 

### （2）sendfile on

​	——零拷贝设置！

​	设置为 on 则**开启 Linux 系统的零拷贝机制**,否则不启用零拷贝。当然,开启后是否起作用,要看所使用的系统版本。CentOS6 及其以上版本支持 sendfile 零拷贝。 

### （3）tcp_nopush on

​	——同一个请求，响应体是否包含响应头信息！

* on:以单独的数据包形式发送 Nginx 的响应头信息,而真正的响应体数据会再以数据包的形式发送,这个数据包中就不再包含响应头信息了。
* off:默认值,响应头信息包含在每一个响应体数据包中。

### （4）tcp_nodelay on

​	——数据发送缓存设置

* on:不设置数据发送缓存,即不推迟发送,适合于传输小数据,无需缓存。
* off:开启发送缓存。若传输的数据是图片等大数据量文件,则建议设置为 off。

### （5）keepalive_timeout 60

​	设置客户端与 Nginx 间所建立的长连接的生命超时时间,时间到达,则连接将自动关闭，单位秒。 

### （6）keepalive_requests 10000

​	设置一个长连接最多可以发送的请求数。该值需要在真实环境下测试。

### （7）client_body_timeout 10

​	设置客户端获取 Nginx 响应的超时时限,即一个请求从客户端发出到接收到 Nginx 的响应的最长时间间隔。若超时,则认为本次请求失败。

# 2.2 请求定位

## 2.2.1 资源访问

### （1）修改配置文件

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1f1n13e4vj30rm0ccae2.jpg)

### （2）创建目录

```shell
#在真实目录中,必须要在 root 属性指定的目录下存在 location 指定的 URI 路径目录。所以需要在/opt/aaa 下创建 xxx/ooo 目录。 
mkdir -p /opt/aaa/xxx/ooo
```

### （3）创建文件

```shell
#在/opt/aaa/xxx/ooo 目录下新建一个 myfile.txt 文件,文件内容为:this default page。
echo "this default page">/opt/aaa/xxx/ooo/myfile.txt

#再新建一个 hello.txt 文件,文件内容为:hello nginx world
echo "hello nginx world">/opt/aaa/xxx/ooo/hello.txt
```

## 2.2.2 路径匹配优先级

### （1）优先级规则

​	优先级由低到高依次是：**普通匹配 < 长路径匹配 < 正则匹配 < 短路匹配 < 精确匹配**

### （2）普通匹配

* 浏览器地址栏中的访问路径均为如下形式,不变。
  ![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1f20qnwyhj30po04wq3x.jpg)
* 下面的匹配规则是:只要请求是以/xxx 开头的路径就可命中。
  ![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1f22i950rj30d406awf9.jpg)
  ![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1f2341catj30og08g3zn.jpg)

### （3）长路径匹配

​	当一个请求路径既可以与一个长路径相匹配,又可以与一个短路径相匹配时,长路径优先级高。

 ![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1f25gr5fsj30d007k3zr.jpg)

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1f255gijhj30lk05u3ze.jpg)

### （4）正则匹配

​	在正则匹配与普通匹配(长路径匹配也属于普通匹配)均可匹配上时,正则匹配的优先级高。 

#### A、区分大小写的正则匹配

​	"~"开头表示区分大小写到正则匹配。 

* 在长路径匹配与正则匹配间,仍然是正则匹配的优先级要高于长路径匹配的,即使正则匹配的要短于长路径匹配的。 
  ![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1f2939xrkj30eq0cyjtf.jpg)
  ![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1f29glyotj30ms06k0tv.jpg)
* 当请求中的 XXX 写为大写字母,会报 404 找到资源。
  ![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1f2a59vp7j30ky05iq3g.jpg)

#### B、不区分大小写的正则匹配

* "~*"开头表示不区分大小写的正则匹配。
  ![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1f2az1c31j30f40am402.jpg)
  ![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1f2b6tazvj30k605qmy1.jpg)
* 当请求中的 XXX 写为大写字母时,依然可以访问。
  ![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1f2bll4jvj30kw05mjsb.jpg)

#### C、正则匹配补充

* !~和!~*分别为区分大小写不匹配及不区分大小写不匹配的正则

### （5）短路匹配

​	以^~开头的匹配路径称为短路匹配,表示只要匹配上,就不再匹配其它的了,即使是正则匹配也不再匹配了。即其优先级要高于正则匹配的。

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1f2cx5p0kj30gc0h0jug.jpg)

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1f2d8xvmpj30ls05smxs.jpg)

### （6）精确匹配	？

​	"="开头表示精确匹配,其是优先级最高的匹配。

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1f2ev6xlwj30em0jmgpe.jpg)

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1f2fleb1yj30l6062dgi.jpg)

## 2.2.3 缓存配置

​	Nginx 具有很强大的缓存功能,可以对请求的 response 进行缓存,起到类似 CDN 的作用, 甚至有比 CDN 更强大的功能。同时,Nginx 缓存还可以用来“**数据托底**”,即当后台 web 服务器挂掉的时候,Nginx 可以直接将缓存中的托底数据返回给用户。此功能就是 Nginx 实现“**服务降级**”的体现。 

​	Nginx 缓存功能的配置由两部分构成：**全局定义**与**局部定义**。在 ```http{}```模块的全局部分中进行缓存全局定义,在 ```server{}```模块的各个``` location{}```模块中根据业务需求进行缓存局部定义。 

### （1）http{}模块的缓存全局定义

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1f2ijn2quj30yy0aogr0.jpg)

#### A、proxy_cache_path(常用)

​	用于指定 Nginx 缓存的存放路径及相关配置。

> * levels=1:2	存储缓存的文件中（即/usr/local/nginx/cache中），将自动建两级目录，一级目录名称为1个字符，二级目录名称为2个字符，如：/usr/local/nginx/cache/a/ab
> * keys_zone=mycache:10m	存储缓存key的内存大小设置（每次存储缓存的时候，都相应指定一个key存储在内存中）keys_zone是设置缓存key的内存区域大小，1m能存8000个key，所以10m已经不小了；
> * max_size=5g    存储缓存的硬盘大小设置，不设置则没有上限，硬盘易被存满！当缓存达到设置上限时，会清理命中率低的缓存以腾出空间；
> * inactive=2h    缓存有效时间，时间到期，内存中的缓存key和硬盘上的缓存都将一块儿被清理；
> * use_temp_path=off    是否使用临时路径（新开辟一个临时空间用于存储缓存内容，并且定期更新存储到硬盘缓存目录中，但存来存去都是在硬盘上存储，看似临时路径缓存效率高一些，实际很不明显，因此一般不设置临时路径，直接设置off即可）

#### B、proxy_temp_path

​	指定 Nginx 缓存的临时存放目录。若 proxy_cache_path 中的 use_temp_path 设置为了 off, 则该属性可以不指定。 

### （2）location{}模块的缓存局部定义

#### A、 proxy_cache mycache(常用)

​	指定用于存放缓存 key 内存区域名称。其值为 http{}模块中 proxy_cache_path 中的keys_zone 的值。

#### B、 proxy_cache_key (常用)

​	指定 Nginx 生成的缓存key的组成。如： \$host\$request_uri

#### C、 proxy_cache_bypass $arg_age

​	指定是否越过缓存，即不使用缓存。比如请求参数包涵$arg_age，则不从缓存取，直接请求服务端

#### D、 proxy_cache_methods GET HEAD

​	指定客户端请求的哪些提交方法将被缓存,默认为 GET 与 HEAD,但不缓存 POST（即使指定post也不会缓存）。 

#### E、 proxy_no_cache \$aaa

​	指定对本次请求是否不做缓存。只要\$aaa不为0，就不对请求结果缓存。 

#### F、 proxy_cache_purge \$ddd 

​	指定是否清除缓存 key。 

#### G、 proxy_cache_lock on (常用)

​	指定是否采用互斥方式回源。类似分布式锁，互斥锁，高并发时，缓存没数据，则将互斥锁分给一个请求去请求服务端并将数据缓存到磁盘上！

#### H、 proxy_cache_lock_timeout 5s  (常用)

​	指定再次生成回源互斥锁的时限。 

#### I、 proxy_cache_valid 5s

 	对指定的 HTTP 状态码的响应数据进行缓存,并指定缓存时间。默认指定的状态码为200,301,302。（根据实际经验，如果对200进行缓存，时间不要太久，太久易产生脏数据；不过，大部分情况是对40x、500错误码进行设置缓存，以便服务降级；）

![](https://ws4.sinaimg.cn/large/006tKfTcly1g1fe5fpvgij30dl03s3zu.jpg)

#### J、 proxy_cache_use_stale http_404 http_500 

​	设置启用托底缓存的条件。而一旦这里指定了相应的状态码,则前面 proxy_cache_calid 中指定的相应状态码所生成的缓存就变为了“托底缓存”。

#### K、 expires 3m 

​	为请求的静态资源开启浏览器端的缓存。 

### （3）Nginx变量

#### A、自定义变量

​	由于 Nginx 配置文件是 perl 脚本,所以其是可以使用如下方式自定义变量的。

![](https://ws3.sinaimg.cn/large/006tKfTcly1g1fee8n5h5j30bs055my5.jpg)

#### B、内置变量

​	Nginx 中已经内置定义了很多变量,这些变量的意义如下:

| 内置变量              | 备注                                                         |
| --------------------- | ------------------------------------------------------------ |
| $args                 | 请求中的参数;                                                |
| $binary_remote_addr   | 远程地址的二进制表示                                         |
| $body_bytes_sent      | 已发送的消息体字节数                                         |
| $content_length       | HTTP 请求信息里的"Content-Length"                            |
| $content_type         | 请求信息里的"Content-Type"                                   |
| $document_root        | 针对当前请求的根路径设置值                                   |
| $document_uri         | 与$uri 相同                                                  |
| $host                 | 请求信息中的"Host",如果请求中没有 Host 行,则等于设置的服务器名; |
| $http_cookie          | cookie 信息                                                  |
| $http_referer         | 来源地址                                                     |
| $http_user_agent      | 客户端代理信息                                               |
| $http_via             | 最后一个访问服务器的 Ip 地址                                 |
| $http_x_forwarded_for | 相当于网络访问路径。                                         |
| $limit_rate           | 对连接速率的限制                                             |
| $remote_addr          | 客户端地址                                                   |
| $remote_port          | 客户端端口号                                                 |
| $remote_user          | 客户端用户名,认证用                                          |
| $request              | 用户请求信息                                                 |
| $request_body         | 用户请求主题                                                 |
| $request_body_file    | 发往后端对本地文件名称                                       |
| $request_filename     | 当前请求对本地文件路径名                                     |
| $request_method       | 请求对方法，比如"GET"、"POST"等                              |
| $request_uri          | 请求的RUI，带参数                                            |
| $server_addr          | 服务器地址，如果没有用 listen 指明服务器地址,使用这个变量将发起一次系统调用以取得地址(造成资源浪费) |
| $server_name          | 请求到达的服务器名                                           |
| $server_port          | 请求到达的服务器端口号                                       |
| $server_protoco       | 请求的协议版本,"HTTP/1.0"或"HTTP/1.1"                        |
| $uri                  | 请求的 URI,可能和最初的值有不同,比如经过重定向之类的         |

# 2.3 Nginx 日志管理及自动切割

​	对于程序员、运维来说,日志非常得重要。通过日志可以查看到很多请求访问信息,及异常信息。Nginx 也提供了对日志的强大支持。 

## 2.3.1 日志管理范围

​	首先,下面要讲的这些日志相关属性可以配置在任意模块。在不同的模块,记录的是不同请求的日志信息。即,日志记录的请求范围是不同的。Nginx 日志一般可以指定三个范围：**http{}模块范围**、**server{}模块范围**,与 **location{}模块范围**。

### （1）http{}模块范围

​	只要有请求通过 http 协议访问该 Nginx,就会有日志信息写入到这里的日志文件。

![](https://ws1.sinaimg.cn/large/006tKfTcly1g1fggur9muj30ou095whz.jpg)

### （2）server{}模块范围

​	只要有请求访问当前 Server,就会有日志信息写入到这里的日志文件。

![](https://ws1.sinaimg.cn/large/006tKfTcly1g1fghfg8enj30jl09dmzo.jpg)

### （3）location{}模拟范围

​	只要有请求访问当前 location,就会有日志信息写入到这里的日志文件。

![](https://ws3.sinaimg.cn/large/006tKfTcly1g1fgi0makdj30kw05n75z.jpg)

## 2.3.2 日志管理指令

​	下面以 http{}模块下的日志为例来学习 Nginx 日志管理指令。

![](https://ws1.sinaimg.cn/large/006tKfTcly1g1fgizi05tj30p608sq6h.jpg)

​	Nginx 的日志分为两类：**访问日志**与**错误日志**。Nginx 整个系统的默认日志在生成预编译文件 makefile 时就已经默认给配置好了。当然,无论是访问日志还是错误日志,其默认路径与名称在 nginx.conf 中均是可以修改的。在配置文件中不仅定义了日志文件的路径及名称,还定义了日志格式。

![](https://ws3.sinaimg.cn/large/006tKfTcly1g1fgkn2boaj30no0b8dtc.jpg)

### （1）log_format

![](https://ws4.sinaimg.cn/large/006tKfTcly1g1fgm722ilj30ou036gn6.jpg)

​	用于设置访问日志的格式,其后的 main 是为该格式所起的名称,可以任意,而其后面的内容则为具体格式,通过 Nginx 内置变量定义。

### （2）access_log

![](https://ws1.sinaimg.cn/large/006tKfTcly1g1fgn1rybzj30nj01u74y.jpg)

该指令用于设置访问日志。上面的格式包含三个参数:

* 第一个参数是日志的存放路径与日志文件名;
* 第二个参数是日志格式名;
* 第三个参数是日志文件所使用的缓存。不过,即使不指定 buffer,其也会存在默认日志缓存的。
* access_log 还可以跟一个参数 off,用于关闭访问日志,即直接写 access_log off 即可关闭访问日志。 

### （3）error_log

![](https://ws3.sinaimg.cn/large/006tKfTcly1g1fgpriyvjj30il03b3zv.jpg)

该指令用于指定错误日志的路径与文件名。需要注意以下几点:

* 其不能指定格式,因为其有默认格式。
* 可以使用自己指定的错误日志文件,不过,将来的访问异常日志就不会再写入到默认的logs/error.log 文件中了。所以关于错误日志,一般使用默认的即可。 
* 错误日志级别由低到高有`:[debug | info | notice | warn | error | crit | alert | emerg]`,默认为 error,级别越高记录的信息越少。
* 错误日志默认是开启的。关闭错误日志的写法为 `error_log /dev/null`; 

### （4）open_log_file_cache

![](https://ws2.sinaimg.cn/large/006tKfTcly1g1fgs7uw5hj30nm01b0ta.jpg)

​	该指令用于打开日志文件读缓存,将日志信息读取到缓存中,以加快对日志的访问。该功能默认为 off,即 `open_log_file_cache off`;

## 2.3.3 默认的/favicon.ico 请求

​	客户端对于服务端页面会自动提交一个/favicon.ico 请求,若没有 favicon.ico 文件则会在日志文件中报出 404。 

​	从网上任意下载一个 ico 图标,将其重命名为 favicon.ico,然后放到 Linux 中的任意目录。 然后再修改 nginx.conf 文件,在其中添加如下的 location{}模块。注意,若将其添加到的位置与页面在同一个目录,下面的 location{}模块不用设置。 

![](https://ws2.sinaimg.cn/large/006tKfTcly1g1fgtokysjj30d305eq3v.jpg)

## 2.3.4 日志自动切割

​	这里仅仅简单介绍一下日志切割的实现步骤,具体实现,网上非常多,用时再查即可。

### （1）创建切割 shell 脚本文件

​	在 Linux 下创建一个实现日志切割的 shell 脚本文件,脚本文件的具体内容可以从网上查找,资源很多。例如,将该 shell 文件创建在 Nginx 安装目录下的 logs 目录中,并命名为cut_nginx_log.sh。

### （2）为该文件添加可执行权限

​	该文件是要作为定时任务被执行的,所以该文件需要具有可执行权限。

### （3）向 crontab 中添加一个定时任务

​	crontab 是 Linux 中的一个定时任务文件,每一行都代表一项定义任务。每行由 6 个字段 组成,前 5 段是时间设定段,第 6 段是任务段。具体格式如下: minute(0-59) hour(0-23) day(1-31) month(1-12) week(0-6) command 

​	本例执行如下命令后会打开文本编辑器,然后再输入如下文本内容即可。

![](https://ws1.sinaimg.cn/large/006tKfTcly1g1fgwfg38ij30ha042dgm.jpg)

![](https://ws4.sinaimg.cn/large/006tKfTcly1g1fgwqrx1dj30p104labs.jpg)

# 2.4 静态代理

​	Nginx 静态代理是指,将所有的静态资源,例如,css、js、html、jpg 等资源存放到 Nginx服务器,而不存放在应用服务器 Tomcat 中。当客户端发出的请求是对这些静态资源的请求时,Nginx 直接将这些静态资源响应给客户端,而无需提交给应用服务器处理。这样就减轻了应用服务器的压力。

## 2.4.1 扩展名拦截

### （1）修改配置文件

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1ga834wvqj30sm0jqn39.jpg)

### （2）创建目录

```shell
#在/opt 目录中创建 statics 目录。
mkdir /opt/statics

#在/opt/statics 目录中创建 css、js、images 目录。
mkdir css js images
```

### （3）上传图片

​	向/opt/statics/images 目录中上传一个图片 car.jpg。

### （4）重启Nginx

```shell
nginx -s reload
```

### （5）浏览器访问

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1gacusiexj30u60bs48v.jpg)

## 2.4.2 目录名拦截

### （1）修改配置文件

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1gae8d62jj30q806wmzk.jpg)

### （2）重启Nginx

```shell
nginx -s reload
```

### （3）浏览器访问

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1gacusiexj30u60bs48v.jpg)

## 2.4.3 页面压缩

### （1）浏览器常见的压缩协议

> * deflate：是一种过时的压缩算法,是 huffman 编码的一种加强。 
>* gzip：是目前大多数浏览器都支持的一种压缩算法,是对 deflate 的改进。 
> * sdch：谷歌开发的一种压缩算法,一种全新的压缩思路。deflate 与 gzip 的压缩思想是：修改传输数据的编码格式以达到减少体量的目的,其最终传输的数据并没有减少。而 sdch 压缩算法的思想是：让冗余的数据仅出现一次,其最终传输的数据减少了。 
>* Zopfli：谷歌开发的一种压缩算法,Deflate 压缩算法的改进。比标准的gzip-9要小 3%-8%, 但压缩用时是 gzip -9 的 80 多倍。 
> * br：即 Brotli,谷歌开发的一种压缩算法,是一种全新的数据格式。与 Zopfli 相比,压缩率能够降低20%-26%。Brotli -1 有着与 Gzip -9 相近的压缩比和更快的压缩解压速度。 
>

### （2）常用设置

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1gapw156jj30ww0b6gpj.jpg)

#### A、 gzip on;

​	开启 gzip 压缩,默认为 off。 

#### B、 gzip_min_length 5k; 

​	指定最小启用压缩的文件大小。 

#### C、 gzip_comp_level 4;

​	指定压缩级别,取值为 1-9,数字越大,压缩比越高,但压缩所用时间会越长。默认为1,建议使用 4。

#### D、 gzip_buffers 4 16k; 

​	“4”表示的是缓存颗粒数量,而“16k”表示的是缓存颗粒大小。 (4的设置一般参考woker数量，16的设置参考最大文件大小)

#### E、 gzip_vary on; 

​	开启动态压缩。默认值 off。 

#### F、 gzip_types mimeType;

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1gaqlzhdgj30x201ajsg.jpg)

​	具体有哪些类型，通过文件```/usr/local/nginx/conf/mime.types```可查看！通过 MIME 类型来指定要压缩的文件类型。默认值 text/html。(也可以压缩图片、css、js，不过图片压缩比小，css、js压缩比高一些)

# 2.5 反向代理

​	通过在 location{}中添加通行代理 proxy_pass 可以指定当前 Nginx 所要代理的真正服务器。

比如,百度代理：

```shell
location / {
            proxy_pass https://www.baidu.com;
        }
```



## 2.5.1 反向代理tomcat服务器

### （1）定义一个web工程

​	定义一个 Maven Web 工程,并命名为 webdemo。其包含一个 JSP 页面,及一个 Servlet。

#### A、修改pom.xml

```xml
	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<maven.compiler.source>1.8</maven.compiler.source>
		<maven.compiler.target>1.8</maven.compiler.target>
	</properties>

	<dependencies>
		<dependency>
			<!-- Servlet 依赖 -->
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.1.0</version>
		</dependency>
		<dependency>
			<!-- JSP 依赖 -->
			<groupId>javax.servlet.jsp</groupId>
			<artifactId>javax.servlet.jsp-api</artifactId>
			<version>2.3.1</version>
		</dependency>
	</dependencies>
```

#### B、定义index.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8"%>
<html>
<head>

</head>
<title>webdemo</title>
<body>
	Nginx World Welcome You!
	<br> Nginx Addr =${pageContext.request.remoteAddr}
	<br> Tomcat Addr =${pageContext.request.localAddr}
	<br>

</body>
</html>
```

#### C、定义SomeServlet

```java
@WebServlet("/some")
public class SomeServlet extends HttpServlet {
	protected void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		PrintWriter writer = response.getWriter();
		writer.println("NginxIp = " + request.getRemoteAddr());
		writer.println("TomcatIp = " + request.getLocalAddr());
	}
}
```

### （2）克隆一台Tomcat并部署工程

​	克隆一台 Tomcat 主机,将 webdemo 工程打包后部署到 Tomcat 的 webapps/ROOT 目录中。当然,需要首先将 ROOT 目录中的文件全部删除。 

### （3）修改Nginx配置文件

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1gcmlnts3j30yi0b8ae5.jpg)

### （4）访问

​	通过 Nginx 反向代理访问 Tomcat 主机的 index.jsp 页面

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1hce370u8j30l60eu0xm.jpg)

​	通过 Nginx 反向代理访问 Tomcat 主机的 SomeServlet。

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1hcerywt4j30kc0e8jvh.jpg)

## 2.5.2 反向代理的属性设置

```shell
#Nginx 允许客户端请求的单文件最大大小,单位字节。(起到一个保护作用)
client_max_body_size 100k; 
#Nginx 为客户端请求设置的缓存大小。
client_body_buffer_size 80k; 

#开启从后端被代理服务器的相应内容缓存区。默认值on。
proxy_buffering on;
#该指令用于设置缓冲区的数量与大小。从被代理的后端服务器取得的响应内容,会缓存到这里。 
proxy_buffers 4 8k; 
#高负荷下缓存大小,其默认值为一般为单个 proxy_buffers 的 2 倍。 
proxy_busy_buffers_size 16k; 

#Nginx 跟后端服务器连接超时时间。默认 60 秒。 
proxy_connect_timeout 60s;
#Nginx 发出请求后等待后端服务器响应的最长时限。默认 60 秒。
proxy_read_timeout 60s;
```

 	

# 2.6 负载均衡

## 2.6.1 负载均衡简介

​	负载均衡,Load Balancing,就是将对请求的处理分摊到多个操作单元上进行。

## 2.6.2 负载均衡分类

### （1）软硬件分类

#### A、硬件负载均衡

​	硬件负载均衡器的性能稳定,且有生产厂商作为专业的服务团队。但其成本很高,一台硬件负载均衡器的价格一般都在十几万到几十万,甚至上百万。知名的负载均衡器有 F5、Array、深信服、梭子鱼等。

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1hcs874cuj313e0d6gw0.jpg)

#### B、软件负载均衡

​	软件负载均衡成本几乎为零,基本都是开源软件。例如,LVS、HAProxy、Nginx 等。

### （2）负载均衡工作层分类

​	负载均衡就其所工作的 OSI 层次,在生产应用层面分为两类:**七层负载均衡**与**四层负载均衡**。当然,为四层负载均衡提供更为底层实现的,还有三层负载均衡与二层负载均衡。

* ==七层负载均衡==：应用层,基于 HTTP 协议,通过虚拟 URL 将请求分配到真实服务器。一般应用于 B/S 架构系统。Nginx 就是七层负载均衡。
* ==四层负载均衡==：传输层,基于 TCP 协议,通过“虚拟 IP + 端口号” 将请求分配到真实服务器。一般应用于 C /S 架构系统。例如,LVS、F5、Nginx Plus(Nginx的商业版) 都属于四层负载均衡。
* 三层负载均衡：网络层,基于 IP 协议,通过虚拟 IP 将请求分配到真实服务器。
* 二层负载均衡：链路层,基于虚拟 MAC 地址将请求分配到真实服务器。  

## 2.6.3 负载均衡的实现

### （1）总体规划

​	该机群包含一台 Nginx 服务器,两台 Tomcat 服务器。将前面打过包的 web 工程直接部署到两台 Tomcat 主机上。然后,在 Nginx 服务器上设置对这两台 Tomcat 主机的负载均衡。

### （2）配置Nginx主机

#### A、修改Nginx主机的hosts文件

```shell
vim /etc/hosts
```

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1hdmj7pfpj30i40a8tbd.jpg)

#### B、修改Nginx配置文件

```shell
weight:权重配置，非优先级，而是按比例分配
```



![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1hdn8s584j30t20nqaj8.jpg)

#### C、配DNS

```shell
192.168.36.114 www.asset1.com
```

## 2.6.4 Nginx负载均衡策略

​	Nginx 内置了三种负载均衡策略,另外,其还支持第三方的负载均衡。而每种负载均衡主机根据负载均衡策略的不同,又可设置很多性能相关的属性。 

### （1）轮询

​	默认的负载均衡策略,其是按照各个主机的**权重比例**依次进行请求分配的。

```shell
upstream tomcat.abc.com {
        server tomcat1:8080 weight=2 fail_timeout=20 max_fail=3;
        server tomcat2:8080 weight=2 fail_timeout=20 max_fail=3;
        server tomcat3:8080 backup;
        server tomcat4:8080 down;
  }
```



![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1hdq2i1h4j312y08iwj3.jpg)

* backup:表示当前服务器为备用服务器（处于启动状态，当tomcat1、tomcat2两台机器有一台机器宕机了，该机器自动接收任务）。
* down:表示当前服务器永久停机。
* fail_ timeout:表示当前主机被 Nginx 认定为停机的最长失联时间,默认为 10 秒。常与 max_fails 联合使用。 
* max_fails:表示在 fail_timeout 时间内最多允许的失败次数。 

### （2）ip_hash

​	指定负载均衡器按照基于**客户端 IP** 的分配方式。

 ![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1he0i5tmuj310007g13d.jpg)

​	对于该策略需要注意以下几点:

* 在 nginx1.3.1 版本之前,该策略中不能指定 weight 属性。 
* 该策略不能与 backup 同时使用。 
* **此策略适合有状态服务,比如 session**。 
* 当有服务器宕机,必须手动指定 down 属性,否则请求仍是会落到该服务器。 

### （3）least_conn

​	把请求转发给连接数最少的服务器。

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1he1pzmtgj312c09gdlf.jpg) 

## 2.6.5 Nginx Plux的四层负载均衡实现

​	补充：[nginx官网](http://nginx.org)，[nginx plux官网](https://www.nginx.com)

​	同样是修改 nginx.conf 文件,添加一个 stream 模块,其与 events、http 等模块同级。在其中配置 upstream{}与 server{}模块。此时需要注意：通行代理配置在 server{}中(前面的是配置在 server 模块的 location 模块中),且不能再是 http://开头的了,因为其负载均衡协议不再是 HTTP 协议了。

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1he3bwb0mj30pa0pkafq.jpg)

# 2.7 动静分离

## 2.7.1 Nginx动静分离的实现

​	下面要搭建的 Nginx 环境中有三台 Nginx 主机:一台用于完成负载均衡,两台用于存放项目中的静态资源。另外,还包含前面的两台 Tomcat 主机。

### （1）修改前面的web工程

​	在前面的 web 工程的 jsp 页面中添加一个背景图片,无需在工程中添加 images 目录及 bj.jpg 图片。因为这些都是要存放在 Nginx 静态代理服务器的。 

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1he9wzd9fj314e0cy441.jpg)

### （2）复制并配置一台 Nginx 服务器

#### A、 修改 nginx.conf

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1heb5yeb8j30sg0b878f.jpg)

#### B、 添加静态资源

​	在/opt/statics 下创建 images 目录,并将 bj.jpg 存放到该目录。

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1hebsfjzqj312c0a2tck.jpg)

### （3）再复制一台 Nginx 服务器

​	以 nginxOS1 为母机,再复制一台 Nginx 主机,并命名为 nginxOS2。完成以下配置:

* 修改主机名:/etc/hostname
* 修改网络配置:/etc/sysconfig/network-scripts/ifcfg-ens33

### （4）修改负载均衡Nginx主机

​	在 nginxOS 主机的 nginx.conf 文件中添加静态代理的负载均衡配置

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1heeuc1q8j30t80ukaml.jpg)

## 2.7.2 项目的启动

### （1）启动两台Tomcat服务器

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1heh98e1qj30q209m44w.jpg)

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1hehg8f6tj30py09kgs0.jpg)

### （2）启动两台Nginx静态代理服务器

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1hehshxhgj30wu06qdib.jpg)

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1hei1v7xjj30xk06o0v7.jpg)

### （3）启动Nginx负载均衡服务器

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1heics0ezj30wa06c76q.jpg)

# 2.8 虚拟主机

## 2.8.1 总体规划

​	现在很多生活服务类网络平台都具有这样的功能:不同城市的用户可以打开不同城市专属的站点。用户首先打开的是平台总的站点,然后允许用户切换到不同的城市。其实,不同的城市都是一个不同的站点。 

​	这里我们要实现的功能是为平台总站点、北京、上海两个城市站点分别创建一个虚拟主 机。每一个虚拟主机都具有两台 Tomcat 的负载均衡主机。由于有三个站点,所以共需六台 Tomcat 主机,克隆 Tomcat 主机太过麻烦,所以这六台 Tomcat 我们使用一台主机实现。在一台主机中安装六个 Tomcat,它们分别使用六个不同的端口号。 

​	首先要创建一个 web 工程,其中就一个 index.jsp 页面,页面除了显示当前城市外,还要显示城市切换的超链接。为了能够再明显的区分出当前访问的 Tomcat,再在页面中显示出当前工程所在的主机名与端口号。 

## 2.8.2 创建三个web工程

​	直接复制前面的 web 工程,每个工程都仅需一个 JSP 即可。最终打为三个 war 包。三个工程的 JSP 页面内容各不相同。

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1hgwqb325j30aq09gjsj.jpg)

### （1）总站

​	注意：JSP 文件的 EL 中通过 request 获取的是 localAddr 与 localPort。

```jsp
<%@ page contentType="text/html;charset=UTF-8"%>
<html>
<head>
<title>entry68</title>
</head>
<body>
	这是 68 平台总站
	<br> 站点切换:
	<a href="http://bj.68.com">北京</a>
	<a href="http://sh.68.com">上海</a>
	<br> Tomcat Addr =${pageContext.request.localAddr}
	<br> Tomcat Port =${pageContext.request.localPort}
	<br>
</body>
</html>
```

### （2）北京站

```jsp
<%@ page contentType="text/html;charset=UTF-8"%>
<html>
<head>
<title>entry68</title>
</head>
<body>
	这是 68 平台［北京］站
	<br> 站点切换:
	<a href="http://www.68.com">总站</a>
	<a href="http://sh.68.com">上海</a>
	<br> Tomcat Addr =${pageContext.request.localAddr}
	<br> Tomcat Port =${pageContext.request.localPort}
	<br>
</body>
</html>
```

### （3）上海站

```jsp
<%@ page contentType="text/html;charset=UTF-8"%>
<html>
<head>
<title>entry68</title>
</head>
<body>
	这是 68 平台［上海］站
	<br> 站点切换:
	<a href="http://www.68.com">总站</a>
	<a href="http://bj.68.com">北京</a>
	<br> Tomcat Addr =${pageContext.request.localAddr}
	<br> Tomcat Port =${pageContext.request.localPort}
	<br>
</body>
</html>
```

## 2.8.3 修改hosts文件

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1hh4b6jzuj30ri0kuahy.jpg)

## 2.8.4 配置Tomcat主机

### （1）项目部署规划

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1hh6419s7j310i0cin7q.jpg)

* 第一、二台 Tomcat 中部署着 entry68.war 包;
* 第三、四台 Tomcat 中部署着 bj68.war 包;
* 第五、六台 Tomcat 中部署着 sh68.war 包。

### （2）配置tomcat主机

​	打开 Tomcat 安装目录下的 conf/server.xml 文件,修改三处端口号。

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1hh75kmo2j311y08otfh.jpg)

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1hh7sf3olj30yk0aiag5.jpg)

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1hh8374z4j3120084q6z.jpg)

## 2.8.5 配置虚拟主机

### （1）直接配置nginx.conf中

​	修改 nginx 的核心配置文件,在其中添加如下内容：

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1hhcjqfw3j30qm0ms47w.jpg)

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1hhdhcsabj30oy0smaj1.jpg)

### （2）单独配置到一个文件

#### A、定义vhosts.conf文件

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1hhelr53qj30nq0s2tgb.jpg)

#### B、修改nginx.conf文件

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1hhf930bqj30po0p411g.jpg)