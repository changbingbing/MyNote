# Spring Boot简介

​	Spring Boot 是由 Pivotal[ˈpɪvətl]团队(一家做大数据的公司) 供的全新框架,其**设计目的**是<u>用来简化新 Spring应用的初始搭建以及开发过程</u>。该框架使用了特定的方式来进行配置,从而使开发人员不再需要定义样板化的配置。通过这种方式,Spring Boot 致力于在蓬勃发展的快速应用开发领域(rapid application development)成为领导者。 

​	简单来说,==SpringBoot 可以简化 Spring 应用程序的开发,使我们不再需要 Spring 配置文件及 web.xml 文件==。 

# Spring Boot工程创建

## 在Idea中创建

### 工程创建

这里要创建一个 Spring Boot 的 Web 工程。首先新建 Spring Initializr。

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g687cw1xy4j314k0qgjy7.jpg)

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g687dbzmirj314q0qkn2p.jpg)

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g687drc74ij315g0ssn5d.jpg)

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g687e0sd83j31580ta0w4.jpg)

### 工程编辑

系统会在前面设置的包中自动生成一个启动类。

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g687g49yxzj314q0fmjx3.jpg)

​	在启动类所在的包下再创建一个子包,在其中编写 SpringMVC 的处理器类。⚠️注意：==要求代码所在的包必须是启动类所在包的子孙包,不能是同级包==。对于本例而言,要求代码必须出现在 com.abc 包的子孙包中。

### 工程运行

​	对于 SpringBoot 程序的运行,若是在 Idea 环境下运行,比较简单,直接运行 main 类即可;若是没有 Idea 环境,则可打包后直接通过 java 命令运行。 

#### Idea下的运行

##### 运行

打开启动类并直接运行即可启动 SpringBoot 框架。在控制台查看启动信息可知:

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g688f07hc4j31360kknaa.jpg)

* Tomcat已启动,且为内置Tomcat,端口号为8080。
* 项目的上下文路径 Context Path,即访问该项目时的项目路径为空,即浏览器访问时无需项目名称。 

##### 访问

在浏览器地址栏中直接输入“主机 + 端口 + URI”即可访问该项目,无需项目名称。

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g688hg6o67j30wo0dqwj5.jpg)

#### 打包后运行

将 SpringBoot 工程打包后即可脱离 Idea 环境运行。

##### 打包

使用 package 命令将工程打为 Jar 包。

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g688i5a5ewj30gs0nq78f.jpg)

##### 运行

​	将打好的 Jar 包移动到任意目录,当然,也可在原来的 target 目录,在命令行即可通过java 命令直接运行。例如,将 jar 包移动到 D:/course 目录中。 

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6cw00pg4oj30og07edh7.jpg)

​	当看到如下 示时,表示应用启动成功。

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6cw1lrncnj312c08aq5k.jpg)

##### 访问

​	在浏览器地址栏中直接输入“主机 + 端口 + URI”即可访问该项目,无需项目名称。

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6cw27dqlpj30vu0ds43d.jpg)

## 官网创建

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6cw38iindj30zs0nuaik.jpg)

​	在点击了 Generate Project 按钮后,即可打开一个下载对话框。官网将配置好的 Spring Boot 工程生成了一个 zip 压缩文件,只要我们将其下载到本地即可。 

​	下载后,将其解压到 idea-workspace 中,在 idea 中即可马上看到该工程。注意⚠️：此时该工程是作为一个 Module 出现的。然后,再通过“导入外部 Moduel 方式”将该工程导入为 Maven 工程即可。 

# 基于war的Spring Boot工程

​	前面创建的 Spring Boot 工程最终被打为了 Jar 包,是以可执行文件的形式出现的,其使 用了 Spring Boot 内嵌的 Tomcat 作为 Web 服务器来运行 web 应用的。新版 Dubbo 的监控中 心工程就是典型的应用。 

​	但在实际生产环境下,<u>对于访问量不大的应用,直接以 Jar 包的形式出现,使用起来是非常方便的,不用部署了。但对于访问量较大的 Web 工程,我们不能使用 Tomcat,而要使 用更为高效的商业 web 容器,例如 JBOSS、WebLogic 等,此时我们需要的是 war 包而非 jar 包</u>。 

​	下面我们来看一下如何使用 Spring Boot 将工程打为 war 包。 

## 工程创建

​	其创建过程与前面打为 jar 包的相似,只以下面的窗口中选择 Packaging 为 War 即可。

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6cw8wh0iuj312s0r6q7g.jpg)

## 工程编辑

​	将第一个工程中的处理器复制到当前工程中即可

## 工程运行

### Idea下运行与访问

打开启动类并直接运行即可启动 SpringBoot 工程,访问方式与之前的也相同。

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6cwbidv2fj30vi0dcwje.jpg)

### 打包部署

#### 打包

​	运行 package 命令,其会打为 war 包。

#### 部署

​	找到该 war 包,将其部署到 Tomcat 的 webapps 目录中,启动 Tomcat。

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6cwcb9sxpj30g60gsjtq.jpg)

#### 访问

​	在浏览器中可以访问到该工程。注意⚠️：于工程是部署到了 Tomcat 的 webapps 中,不是部署到webapps/ROOT 中,所以在访问时需要指定工程名。

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6cwernz18j313u0cgwj7.jpg)

# Spring Boot的主配置文件

## 编辑器

​	Spring Boot 的主配置文件是 src/main/resources 中默认创建的 spring.properties 文件。该文件打开后是没有自动 示功能的。此时可以打开 Project Structure 窗口,在 Modules 中选中没有自动 示的工程,点击+号,找到Spring,将其添加可以。此时的配置文件就有了自动 示功能,包括后面的 yml 文件也有了自动 示。

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6cwk4mjl8j30vj0u0gv7.jpg)

## 简单尝试

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6cwkwnh3dj30qa0amn18.jpg)

​	在 Eclipse 中运行工程后,查看日志文件可以看到端口号与应用的根的确发生的变化。

![image-20190826111615849](/Users/changbingbing/Library/Application Support/typora-user-images/image-20190826111615849.png)

​	在地址栏中需要键入新的端口号与应用的根名称。

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6cwtiadnaj30uu0bogq0.jpg)

​	不过需要注意⚠️：这里指定的 Tomcat 的端口号及应用的根路径,**仅仅**是针对于内置 Tomcat的,是测试时使用的。将工程打为 war 包后部署到真正的 Tomcat,这些配置是不起作用的,即 Tomcat 的端口号为真正 Tomcat 的端口号,而项目的根路径为 war 包名称。

## yml文件

​	Spring Boot 的主配置文件也可使用 application.yml 文件。yml,也可写为 yaml。 

​	在开发之初 YAML 的本意是 Yet Another Markup Language(仍是一种标记语言)。后来为 了强调这种语言是以数据为中心,而不是以标记为中心,所以将 YAML 解释为 Yaml Ain't Markup Language(Yaml 不是一种标记语言)。它是一种直观的能够被电脑识别的数据序列化 格式,是一个可读性高并且容易被人阅读,容易和脚本语言交互,用来表达多级资源序列的编程语言。 

​	<u>yml 与 properties 文件的主要区别是对于多级属性,即 key 的显示方式不同</u>。yml 文件在输入时,只需按照点(.)的方式输出 key 即可,输入完毕后回车即出现了如下形式。该形式要求：**冒号后与值之间要有一个空格。不同级别的属性间要有两个空格的缩进。** 

需要注意⚠️：很多脚本中的空格都是作为无效字符出现的,但 yml 脚本则是作为有效字符出现的,必须要保证空格的数量。 

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6cx7c2rldj312w0jg483.jpg)

​	在演示时需要注意⚠️：**application.properties 与 application.yml 这两个文件只能有一个。要求文件名必须为 application。**所以,此时可以将 application.properties 文件重命名为其它名字即可。

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6cx8lt9s2j30gw06gmyo.jpg)

# Actuator监控器

​	Actuator['æktʃʊˌeɪtə](激励者;执行器)是 Spring Boot  供的一个可挺拔模块,用于对工程进行监控。其通过不同的监控终端实现不同的监控功能。其功能与 Dubbo 的监控中心类似,不同的是,Dubbo 的监控中心是需要专门部署的,而 Spring Boot 的 Actuator 是存在于每一个工程中的。

## 基本环境搭建

​	随便一个 Spring Boot 工程中都可以使用 Actuator 对其进行监控。本例使用 01-primary 工程作为被监控对象,复制该工程并重命名为 03-actuatorTest,在该工程中进行修改。 

### 导入依赖

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
			<!-- <version>2.0.5.RELEASE</version> -->
</dependency>
```

### 修改配置文件

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6cxd3mvnij310e0luak5.jpg)

### 访问测试－health监控终端

启动该工程后在地址栏访问,health 是一种监控终端,用于查看当前应用程序(微服务)的健康状况。

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6cxf0sad2j30yu0ga7ae.jpg)

## 添加info信息

### 修改配置文件

在配置文件中添加如下 Info 信息,则可以通过 info 监控终端查看到。

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6cxhl9t7cj30uh0u0dzh.jpg)

### 访问测试

这些信息都是以 JSON 的格式显示在浏览器的。

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d10opa89j30vy0u0dpn.jpg)

## 开放其它监控终端

​	默认情况下,Actuator 仅开放了 health 与 info 两个监控终端,但其还有很多终端可用, 不过,需要手工开放。 

### 修改配置文件

在配置文件中添加如下内容。

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6d1369txij30xw08ytcv.jpg)

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d13r99x5j312y0lok0p.jpg)

### 访问测试

#### mappings终端

​	通过 mappings 终端,可以看到当前工程中所有的 URI 与处理器的映射关系,及详细的
处理器方法及其映射规则。很实用。

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d18pqpxej312w0ne0yv.jpg)

#### beans终端

可以查看到当前应用中所有的对象信息。

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6d19fz7ewj312k0t8qc5.jpg)

#### env终端

可以看到当前应用程序运行主机的所有软硬件环境信息。

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d1a3j09wj312i0h00z3.jpg)

## 单独关闭某些监控终端

在开放了所有监控终端的情况下,有些终端显示的信息并不想公开,此时可以单独关闭这些终端。 

### 修改配置文件

在配置文件中添加如下内容：

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d1b2aundj312s0e8jyb.jpg)

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6d1bcxuxxj312s0mswp6.jpg)

### 访问测试

在关闭这些终端后,其它终端仍可继续使用。

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d1bui6jnj31060lmdnk.jpg)

## 常用的监控终端

在百度搜索“springboot actuator”即可找到如下表格。

| HTTP 方法 | 监控终端        | 功能描述                                                     |
| :-------- | :-------------- | :----------------------------------------------------------- |
| GET       | /auditevents    | 显示应用暴露的审计事件 (比如认证进入、订单失败)              |
| GET       | /beans          | 描述应用程序上下文里全部的 Bean，以及它们的关系              |
| GET       | /conditions     | 就是 1.0 的 /autoconfig ，提供一份自动配置生效的条件情况，记录哪些自动配置条件通过了，哪些没通过 |
| GET       | /configprops    | 描述配置属性(包含默认值)如何注入Bean                         |
| GET       | /env            | 获取全部环境属性                                             |
| GET       | /env/{name}     | 根据名称获取特定的环境属性值                                 |
| GET       | /flyway         | 提供一份 Flyway 数据库迁移信息                               |
| GET       | /liquidbase     | 显示Liquibase 数据库迁移的纤细信息                           |
| GET       | /health         | 报告应用程序的健康指标，这些值由 HealthIndicator 的实现类提供 |
| GET       | /heapdump       | dump 一份应用的 JVM 堆信息                                   |
| GET       | /httptrace      | 显示HTTP足迹，最近100个HTTP request/repsponse                |
| GET       | /info           | 获取应用程序的定制信息，这些信息由info打头的属性提供         |
| GET       | /logfile        | 返回log file中的内容(如果 logging.file 或者 logging.path 被设置) |
| GET       | /loggers        | 显示和修改配置的loggers                                      |
| GET       | /metrics        | 报告各种应用程序度量信息，比如内存用量和HTTP请求计数         |
| GET       | /metrics/{name} | 报告指定名称的应用程序度量值                                 |
| GET       | /scheduledtasks | 展示应用中的定时任务信息                                     |
| GET       | /sessions       | 如果我们使用了 Spring Session 展示应用中的 HTTP sessions 信息 |
| POST      | /shutdown       | 关闭应用程序，要求endpoints.shutdown.enabled设置为true       |
| GET       | /mappings       | 描述全部的 URI路径，以及它们和控制器(包含Actuator端点)的映射关系 |
| GET       | /threaddump     | 获取线程活动的快照                                           |

# 相关文档推荐：

* [SpringBoot简介](https://www.cnblogs.com/cb0327/p/7675184.html)

