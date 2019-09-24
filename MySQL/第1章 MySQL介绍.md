# 数据库概述

## 什么是数据库

> 数据库就是[存储数据的仓库],其本质是一个[文件系统],数据按照特定的格式将数据存储起来,用户可以通过SQL对数据库中的数据进行增加、修改、删除及查询操作。

## 什么是关系型数据库

> 数据库中的[记录是有行有列的数据库]就是关系型数据库,与之相反的就是NoSQL数据库了。

## 数据库和表

![](https://ws4.sinaimg.cn/large/006tNc79ly1fzfrewinyrj30gz073jsl.jpg)

> 数据库管理系统(DataBase Management System,DBMS):指一种[操作和管理数据库]的大型软件,用于创建、使用和维护数据库,对数据库进行统一管理和控制,以保证数据库的安全性和完整性。用户通过数据库管理系统访问数据库中表内的数据。(记录)

## 常见的数据库管理系统

```shell
- MYSQL :开源免费的数据库,小型的数据库.已经被Oracle收购了。MySQL5.5版本之后都是由Oracle发布 的版本。
- Oracle :收费的大型数据库,Oracle公司的产品。Oracle收购SUN公司,收购MYSQL。
- DB2 :IBM公司的数据库产品,收费的。常应用在银行系统中。在中国的互联网公司,要求去IOE(IBM小型
机、Oracle数据库、EMC存储设备)
- SQLServer:MicroSoft 公司收费的中型的数据库。C#、.net等语言常使用。
- SyBase :已经淡出历史舞台。提供了一个非常专业数据建模的工具PowerDesigner。 - SQLite : 嵌入式的小型数据库,应用在手机端。 
```

这里主要讨论的是：MySQL

# MySQL介绍

## MySQL是什么

> MySQL 是最流行的【关系型数据库管理系统】,在WEB应用方面 MySQL是最好的RDBMS应用软件之一。

## MySQL发展历程

```shell
- MySQL的历史可以追溯到1979年,一个名为Monty Widenius的程序员在为TcX的小公司打工,并且用BASIC 设计了一个报表工具,使其可以在4MHz主频和16KB内存的计算机上运行。当时,这只是一个很底层的且仅面向报表 的存储引擎,名叫Unireg。
- 1990年,TcX公司的客户中开始有人要求为他的API提供SQL支持。Monty直接借助于mSQL的代码,将它集成 到自己的存储引擎中。令人失望的是,效果并不太令人满意,决心自己重写一个SQL支持。
- 1996年,MySQL 1.0发布,它只面向一小拨人,相当于内部发布。
- 到了1996年10月,MySQL 3.11.1发布(MySQL没有2.x版本),最开始只提供Solaris下的二进制版本。一个
月后,Linux版本出现了。在接下来的两年里,MySQL被依次移植到各个平台。
- 【1999~2000年】,【MySQL AB】公司在瑞典成立。Monty雇了几个人与Sleepycat合作,开发出了
【Berkeley DB引擎】, 由于BDB支持事务处理,因此MySQL从此开始支持事务处理了。
- 2000,MySQL不仅公布自己的源代码,并采用GPL(GNU General Public License)许可协议,正式进入开
源世界。同年4月,MySQL对旧的存储引擎ISAM进行了整理,将其命名为MyISAM。
- 2001年,集成Heikki Tuuri的存储引擎【InnoDB】,这个引擎不仅能【支持事务处理,并且支持行级
锁】。后来该引擎被证明是最为成功的MySQL事务存储引擎。【MySQL与InnoDB的正式结合版本是4.0】
- 2003年12月,【MySQL 5.0】版本发布,提供了视图、存储过程等功能。
- 【2008年1月】,【MySQL AB公司被Sun公司以10亿美金收购】,MySQL数据库进入Sun时代。在Sun时代, Sun公司对其进行了大量的推广、优化、Bug修复等工作。
- 2008年11月,MySQL 5.1发布,它提供了分区、事件管理,以及基于行的复制和基于磁盘的NDB集群系统,同 时修复了大量的Bug。
- 【2009年4月】,Oracle公司以74亿美元收购Sun公司,自此MySQL数据库进入Oracle时代,而其第三方的 存储引擎InnoDB早在2005年就被Oracle公司收购。
- 2010年12月,【MySQL 5.5发布】,其主要新特性包括半同步的复制及对SIGNAL/RESIGNAL的异常处理功能 的支持,【最重要的是InnoDB存储引擎终于变为当前MySQL的默认存储引擎】。MySQL 5.5不是时隔两年后的一次 简单的版本更新,而是加强了MySQL各个方面在企业级的特性。Oracle公司同时也承诺MySQL 5.5和未来版本仍是 采用GPL授权的开源产品。
```

# SQL介绍

## 什么是SQL

> ​	【SQL是Structured Query Language的缩写】,它的前身是著名的关系数据库原型系统System R所采用的SEQUEL语言。作为一种访问【关系型数据库的标准语言】,SQL自问世以来得到了广泛的应用,不仅是著名的大型商用数据库产品Oracle、DB2、Sybase、SQL Server支持它,很多开源的数据库产品如PostgreSQL、MySQL也支持它,甚至一些小型的产品如Access也支持SQL。近些年蓬勃发展的NoSQL系统最初是宣称不再需要SQL的,后来也不得不修正为Not Only SQL,来拥抱SQL。
>
> ​	蓝色巨人IBM对关系数据库以及SQL语言的形成和规范化产生了重大的影响,第一个版本的SQL标准SQL86就是基于System R的手册而来的。Oracle在1979年率先推出了支持SQL的商用产品。随着数据库技术和应用的发展,为不同RDBMS提供一致的语言成了一种现实需要。
>
> ​	对SQL标准影响最大的机构自然是那些著名的数据库产商,而具体的制订者则是一些非营利机构,例如【国际标准化组织ISO、美国国家标准委员会ANSI】等。各国通常会按照 ISO标准和ANSI标准(这两个机构的很多标准是差不多等同的)制定自己的国家标准。中国是ISO标准委员会的成员国,也经常翻译一些国际标准对应的中文版。标准为了避免采用具体产品的术语,往往会抽象出很多名词,从而增加了阅读和理解的难度,翻译成中文之后更容易词不达意。对于数据库系统实现者和用户而言,很多时候还不如直接读英文版本为好。虽然正式的标准不像RFC那样可以从网络上免费获得,标准草案还是比较容易找到的(如:http://www.jtc1sc32.org/doc/)。待批准的标准草案和最终的标准也没有什么实质上的区别,能够满足日常工作的需要。	
>
> 下面是SQL发展的简要历史:
> 1986年,ANSI X3.135-1986,ISO/IEC 9075:1986,SQL-86
> 1989年,ANSI X3.135-1989,ISO/IEC 9075:1989,SQL-89
> 1992年,ANSI X3.135-1992,ISO/IEC 9075:1992,SQL-92(SQL2)
> 1999年,ISO/IEC 9075:1999,SQL:1999(SQL3)
> 2003年,ISO/IEC 9075:2003,SQL:2003
> 2008年,ISO/IEC 9075:2008,SQL:2008
> 2011年,ISO/IEC 9075:2011,SQL:2011
>
> ​	如果要了解标准的内容,比较推荐的方法是【泛读SQL92】(因为它涉及了SQL最基础和最核心的一些内容),然后增量式的阅读其他标准。

```shell
不只是mysql还有其他数据库,在SQL92或者SQL99这些国际SQL标准基础之上,它们还扩展了自己的一些SQL语句, 比如MySQL中的limit关键字
```

## SQL的语言分类

```shell
- 数据定义语言:简称【DDL】(Data Definition Language),用来定义数据库对象:数据库,表,列等。关 键字:create,alter,drop等
- 数据操作语言:简称【DML】(Data Manipulation Language),用来对数据库中表的记录进行更新。关键 字:insert,delete,update等
- 数据控制语言:简称【DCL】(Data Control Language),用来定义数据库的访问权限和安全级别,及创建用 户;关键字:grant等
- 数据查询语言:简称【DQL】(Data Query Language),用来查询数据库中表的记录。关键字:select, from,where等
```

