# 3.1 多版本控制

​	灰度发布(金丝雀发布)时使用。新老版本终将不能共存，所以二者不能存在依赖。

## 3.1.1 创建提供者 04-provider-version

### (1) 创建工程

​	复制前面的提供者工程 `02-provider-zk`,并更名为 `04-provider-version`。

### (2) 定义两个接口实现类

​	删除原来的 SomeServiceImpl 类,并新建两个实现类

```java
///04-provider-version/src/main/java/com/abc/provider/OldServiceImpl.java
public class OldServiceImpl implements SomeService {
    @Override
    public String hello(String name) {
        System.out.println("执行【老】的提供者OldServiceImpl的hello() ");
        return "OldServiceImpl";
    }
}
```

```java
///04-provider-version/src/main/java/com/abc/provider/NewServiceImpl.java
public class NewServiceImpl implements SomeService {
    @Override
    public String hello(String name) {
        System.out.println("执行【新】的提供者NewServiceImpl的hello() ");
        return "NewServiceImpl";
    }
}
```

### (3) 修改配置文件

​	指定版本 0.0.1 对应的是 oldService 实例,而版本 0.0.2 对应的是 newService 实例。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <dubbo:application name="04-provider-version"/>

    <dubbo:registry address="zookeeper://zkOS:2181"/>
    

    <!--注册Service实现类-->
    <bean id="oldService" class="com.abc.provider.OldServiceImpl"/>
    <bean id="newService" class="com.abc.provider.NewServiceImpl"/>

    <!--暴露服务-->
    <dubbo:service interface="com.abc.service.SomeService"
                   ref="oldService" version="0.0.1"/>
    <dubbo:service interface="com.abc.service.SomeService"
                   ref="newService" version="0.0.2" protocol="dubbo,rmi"/>

</beans>
```

## 3.1.2 创建消费者 04-consumer-version

### (1) 创建工程

​	复制前面的消费者工程 `02-consumer-zk`,并更名为 `04-consumer-version`。

### (2) 修改配置文件

​	该工程无需修改消费者类 SomeConsumer。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    
    <dubbo:application name="04-consumer-version"/>

    <dubbo:registry address="zookeeper://zkOS:2181" />

    <!--指定消费0.0.1版本，即oldService提供者-->
    <!--<dubbo:reference id="someService"  version="0.0.1" protocol="dubbo"-->
                     <!--interface="com.abc.service.SomeService"/>-->

    <!--指定消费0.0.2版本，即newService提供者-->
    <dubbo:reference id="someService"  version="0.0.2" protocol="rmi"
                     interface="com.abc.service.SomeService"/>

</beans>
```

# 3.2 服务分组

分组实现服务可共存，组间允许存在依赖。

## 3.2.1 创建提供者 05-provider-group

### (1) 创建工程

​	复制前面的提供者工程 `04-provider-version`,并更名为 `05-provider-group`。

### (2) 定义两个接口实现类

​	删除原来的两个接口实现类,重新定义两个新的实现类

```java
///05-provider-group/src/main/java/com/abc/provider/WeixinServiceImpl.java
public class WeixinServiceImpl implements SomeService {
    @Override
    public String hello(String name) {
        System.out.println("使用【微信】付款");
        return "WeixinServiceImpl";
    }
}
```

```java
///05-provider-group/src/main/java/com/abc/provider/ZhifubaoServiceImpl.java
public class ZhifubaoServiceImpl implements SomeService {
    @Override
    public String hello(String name) {
        System.out.println("使用【支付宝】付款");
        return "ZhifubaoServiceImpl";
    }
}
```

### (3) 修改配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <dubbo:application name="05-provider-group"/>

    <dubbo:registry address="zookeeper://zkOS:2181"/>

    <!--声明服务暴露协议-->
    <dubbo:protocol name="dubbo" port="20880"/>
    <dubbo:protocol name="rmi" port="1099"/>

    <!--注册Service实现类-->
    <bean id="weixinService" class="com.abc.provider.WeixinServiceImpl"/>
    <bean id="zhifubaoService" class="com.abc.provider.ZhifubaoServiceImpl"/>

    <!--暴露服务-->
    <dubbo:service interface="com.abc.service.SomeService"
                   ref="weixinService" group="pay.weixin" protocol="dubbo"/>
    <dubbo:service interface="com.abc.service.SomeService"
                   ref="zhifubaoService" group="pay.zhifubao" protocol="rmi"/>
</beans>
```

## 3.2.2 创建消费者 05-consumer-group

### (1) 创建工程

​	复制前面的提供者工程 `04-consumer-version`,并更名为 `05-consumer-group`。

### (2) 修改配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    
    <dubbo:application name="05-consumer-group"/>

    <dubbo:registry address="zookeeper://zkOS:2181" />

    <!--指定调用微信服务，指定使用dubbo协议连接提供者-->
    <dubbo:reference id="weixin"  group="pay.weixin" protocol="dubbo"
                     interface="com.abc.service.SomeService"/>
    <!--指定调用支付宝服务，指定使用rmi协议连接提供者-->
    <dubbo:reference id="zhifubao"  group="pay.zhifubao" protocol="rmi"
                     interface="com.abc.service.SomeService"/>

</beans>
```

### (3) 修改消费者类

```java
public class ConsumerRun {
    public static void main(String[] args) {
        ApplicationContext ac = new ClassPathXmlApplicationContext("spring-consumer.xml");

        // 使用微信支付
        SomeService weixinService = (SomeService) ac.getBean("weixin");
        String weixin = weixinService.hello("China");
        System.out.println(weixin);

        // 使用支付宝支付
        SomeService zhifubaoService = (SomeService) ac.getBean("zhifubao");
        String zhifubao = zhifubaoService.hello("China");
        System.out.println(zhifubao);
    }
}
```

# 3.3 多协议支持

## 3.3.1 服务暴露协议

​	我们在前面的示例中,服务提供者与服务消费者都是通过 zookeeper 连接协议连接上Zookeeper 注册中心的。 

```xml
 <dubbo:registry address="zookeeper://zkOS:2181" />
```

​	提供者与消费者均连接上了注册中心,那么消费者就理所当然的可以享受提供者提供的服务了?

​	实际情况并不是这样的。前述的 zookeeper 协议,是消费者/提供者连接注册中心的连接协议,而非消费者与提供者间的连接协议。

​	当消费者连接上注册中心后,在消费服务之前,首先需要连接上这个服务提供者。虽然消费者通过注册中心可以获取到服务提供者,但提供者对于消费者来说却不是透明的,消费者并不知道真正的服务提供者是谁。不过,无论提供者是谁,**消费者都必须连接上提供者才可以获取到真正的服务**,而这个连接也是需要**专门的连接协议**的。这个协议称为**服务暴露协议**。

​	可是我们之前的代码中并没有看到服务暴露协议的相关配置,但仍可正常运行项目,这是为什么呢?因为**采用了默认的暴露协议:Dubbo 服务暴露协议**。除了 **==Dubbo 服务暴露协议==**外,Dubbo 框架还支持另外 8 种服务暴露议:`RMI 协议`、`Hessian 协议`、`HTTP 协议`、`WebService 协议`、`Thrift 协议`、`Memcached 协议`、`Redis 协议`、`Rest 协议`。但在实际生产中,<u>使用最多的就是 Dubbo 服务暴露协议</u>。

​	对于多协议的用法有两种：一种是**同一个服务支持多种协议**,一种是**不同的服务使用不同的协议**。首先来看“同一服务支持多种协议”的用法。

## 3.3.2 各个协议特点

### (1) dubbo 协议

> * **Dubbo 默认传输协议** 
> * 连接个数:单连接 
> * 连接方式:长连接 （连接数据传输完毕，连接还在；再次传输数据，依然用这个连接；并发量高、数据量小则适用长连接）
> * 传输协议:TCP 
> * 传输方式:NIO 异步、非阻塞传输 
> * 适用范围:传入传出参数数据包较小(建议小于 100K),**消费者比提供者个数多**,单一消费者无法压满提供者,尽量**不要用 dubbo 协议传输大文件或超大字符串**。 

### (2) rmi 协议

> * 采用 JDK 标准的java.rmi.*实现 
> * 连接个数:多连接
> * 连接方式:短连接（一个连接数据传输完毕，则连接断开消失；再有数据传输，则在此创建连接；数据量大则适用短连接）
> * 传输协议:TCP 
> * 传输方式:BIO 同步传输
> * 适用范围:传入传出参数数据包大小混合,**消费者与提供者个数差不多**,可传文件。 

### (3) hession 协议

> * 连接个数:多连接 
> * 连接方式:短连接 
> * 传输协议:HTTP 
> * 传输方式:BIO 同步传输 
> * 适用范围:传入传出参数**数据包较大**,**提供者比消费者个数多**,提供者抗压能力较大, 可传文件

### (4) http 协议 

> * 连接个数:多连接 
> * 连接方式:短连接 
> * 传输协议:HTTP 
> * 传输方式:BIO 同步传输 
> * 适用范围:传入传出参数数据包大小混合,**提供者比消费者个数多**；可用浏览器查看, 可用表单或 URL 传入参数,暂不支持传文件。
> * 较hession协议优势：可以直接使用表单，也可以直接在url中传参来传输数据，而hession不行；另外，使用http协议，可以直接用浏览器查看。

### (5) webService 协议 

> * 连接个数:多连接
> * 连接方式:短连接
> * 传输协议:HTTP
> * 传输方式:BIO 同步传输
> * 适用场景:系统集成、跨语言调用 

### (6) thrift 协议 

> ​	Thrift 是 Facebook 捐给 Apache 的一个 RPC 框架,其消息传递采用的协议即为 thrift 协议。当前 dubbo 支持的 thrift 协议是对 thrift 原生协议的扩展。Thrift 协议不支持 null 值的传递。 

### (7) memcached 协议与 redis 协议

​	它们都是**高效的 KV 缓存服务器**。它们会对传输的数据使用相应的技术进行缓存。

### (8) rest 协议

​	若需要开发具有 RESTful 风格的服务,则需要使用该协议。

## 3.3.3 同一服务支持多种协议

### (1) 修改提供者配置文件

​	在提供者中要首先声明新添加的协议,然后在服务`<dubbo:service/>`标签中再增加该新的协议。若不指定,默认为 dubbo 协议。	

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <dubbo:application name="04-provider-version"/>

    <dubbo:registry address="zookeeper://192.168.0.104:2181"/>

    <!--声明服务暴露协议-->
    <dubbo:protocol name="dubbo" port="20880"/>
    <dubbo:protocol name="rmi" port="1099"/>


    <!--注册Service实现类-->
    <bean id="oldService" class="com.abc.provider.OldServiceImpl"/>
    <bean id="newService" class="com.abc.provider.NewServiceImpl"/>

    <!--暴露服务-->
    <dubbo:service interface="com.abc.service.SomeService"
                   ref="oldService" version="0.0.1"/>
    <dubbo:service interface="com.abc.service.SomeService"
                   ref="newService" version="0.0.2" protocol="dubbo,rmi"/>

</beans>
```

​	这里需要理解这个服务暴露协议的意义。其是指出,消费者若要连接当前的服务,就需要通过这里指定的协议及端口号进行访问。这里的端口号可以是任意的,不一定非要使用默认的端口号(Dubbo 默认为 20880,rmi 默认为 1099)。这里指定的协议名称及端口号,在当前服务注册到注册中心时会一并写入到服务映射表中。当消费者根据服务名称查找到相应主机时,其同时会查询出消费此服务的协议、端口号等信息。其底层就是一个 **Socket 编程**,通过主机名与端口号进行连接。

### (2) 修改消费者配置文件

​	在消费者引用服务时要指出所要使用的协议。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    
    <dubbo:application name="04-consumer-version"/>

    <dubbo:registry address="zookeeper://zkOS:2181" />

    <!--指定消费0.0.1版本，即oldService提供者-->
    <!--<dubbo:reference id="someService"  version="0.0.1" protocol="dubbo"-->
                     <!--interface="com.abc.service.SomeService"/>-->

    <!--指定消费0.0.2版本，即newService提供者-->
    <dubbo:reference id="someService"  version="0.0.2" protocol="rmi"
                     interface="com.abc.service.SomeService"/>

</beans>
```

## 3.3.4 不同服务使用不同协议

### (1) 修改提供者配置文件

​	首先要修改服务提供者配置文件:声明所要使用的协议;在使用`<dubbo:service/>`暴露服务时通过 protocal 属性指定所要使用的服务协议。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <dubbo:application name="05-provider-group"/>

    <dubbo:registry address="zookeeper://zkOS:2181"/>

    <!--声明服务暴露协议-->
    <dubbo:protocol name="dubbo" port="20880"/>
    <dubbo:protocol name="rmi" port="1099"/>

    <!--注册Service实现类-->
    <bean id="weixinService" class="com.abc.provider.WeixinServiceImpl"/>
    <bean id="zhifubaoService" class="com.abc.provider.ZhifubaoServiceImpl"/>

    <!--暴露服务-->
    <dubbo:service interface="com.abc.service.SomeService"
                   ref="weixinService" group="pay.weixin" protocol="dubbo"/>
    <dubbo:service interface="com.abc.service.SomeService"
                   ref="zhifubaoService" group="pay.zhifubao" protocol="rmi"/>
</beans>
```

### (2) 修改消费者配置文件

​	然后在消费者端通过`<dubbo:reference/>`引用服务时通过添加 protocal 属性指定要使用
的服务协议。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    
    <dubbo:application name="05-consumer-group"/>

    <dubbo:registry address="zookeeper://zkOS:2181" />

    <!--指定调用微信服务，指定使用dubbo协议连接提供者-->
    <dubbo:reference id="weixin"  group="pay.weixin" protocol="dubbo"
                     interface="com.abc.service.SomeService"/>
    <!--指定调用支付宝服务，指定使用rmi协议连接提供者-->
    <dubbo:reference id="zhifubao"  group="pay.zhifubao" protocol="rmi"
                     interface="com.abc.service.SomeService"/>

</beans>
```

# 3.4 负载均衡

## 3.4.1 搭建负载均衡环境

​	该负载均衡中同一服务会有三个提供者,它们均复制于原来的 `02-provider-zk`。消费者不用变。 

### (1) 复制提供者 02-provider-zk01

#### A、 创建工程

​	复制前面的 `02-provider-zk` 工程,并重命名为 `02-provider-zk01`。

#### B、 修改配置文件

​	由于这三个提供者工程都运行在当前主机,它们提供的服务名称相同(均为接口名),且均使用默认的 Dubbo 服务暴露协议,所以必须要修改这三个提供者 Dubbo 的默认端口号,否则会发生端口号冲突。但在生产环境中,这些服务均运行在不同的服务器上,是不会出现Dubbo 端口号冲突的问题的。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <dubbo:application name="02-provider-zk01"/>

    <dubbo:protocol name="dubbo" port="20881"/>

    <!--指定服务注册中心：zk单机-->
    <dubbo:registry address="zookeeper://zkOS:2181"/>

    <bean id="someService" class="com.abc.provider.SomeServiceImpl"/>

    <dubbo:service interface="com.abc.service.SomeService"
                   ref="someService"/>

</beans>
```

#### C、 修改 Service 实现类

​	为了在运行时可以看到负载均衡效果,即每次请求访问到的都是不同的提供者工程,这里修改一个 Service 实现类中业务方法的返回值。 

```java
public class SomeServiceImpl implements SomeService {
    @Override
    public String hello(String name) {
        System.out.println("执行【第一个】提供者的hello() ");
        return "【第一个】提供者";
    }
}
```

### (2) 复制提供者 02-provider-zk02

#### A、 创建工程

​	复制前面的 `02-provider-zk01` 工程,并重命名为 `02-provider-zk02`。

#### B、 修改配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <dubbo:application name="02-provider-zk01"/>

    <dubbo:protocol name="dubbo" port="20882"/>

    <!--指定服务注册中心：zk单机-->
    <dubbo:registry address="zookeeper://zkOS:2181"/>

    <bean id="someService" class="com.abc.provider.SomeServiceImpl"/>

    <dubbo:service interface="com.abc.service.SomeService"
                   ref="someService"/>

</beans>
```

#### C、 修改 Service 实现类

```java
public class SomeServiceImpl implements SomeService {
    @Override
    public String hello(String name) {
        System.out.println("执行【第二个】提供者的hello() ");
        return "【第二个】提供者";
    }
}
```

### (3) 复制提供者 02-provider-zk03

#### A、 创建工程

​	复制前面的 `02-provider-zk01` 工程,并重命名为 `02-provider-zk03`。

#### B、 修改配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <dubbo:application name="02-provider-zk01"/>

    <dubbo:protocol name="dubbo" port="20883"/>

    <!--指定服务注册中心：zk单机-->
    <dubbo:registry address="zookeeper://zkOS:2181"/>

    <bean id="someService" class="com.abc.provider.SomeServiceImpl"/>

    <dubbo:service interface="com.abc.service.SomeService"
                   ref="someService"/>

</beans>
```

#### C、 修改 Service 实现类

```java
public class SomeServiceImpl implements SomeService {
    @Override
    public String hello(String name) {
        System.out.println("执行【第三个】提供者的hello() ");
        return "【第三个】提供者";
    }
}
```

## 3.4.2 负载均衡算法

### (1) Dubbo 内置的负载均衡算法

> Dubbo 内置了四种负载均衡算法。
>
> * **random**:随机算法,是**默认**的负载均衡策略。
> * **roundrobin**:轮询算法。按照权重进行访问。**权重设置在提供者端**（数值越大,权重越大）。
> * **leastactive**:最少活跃度算法（谁被调用的次数越少，优先级越高，刚开始默认采用轮询）。
> * **consistenthash**:一致性 hash 算法（相同的参数［单个参数］请求落到相同的提供者上；如果是多个参数，则只对第一个参数进行hash算法）。 
>
> 思考：如果请求落到2号提供者上，刚好2号提供者宕机了，会怎样？
>
> 答：Dubbo有个机制，如果请求落到2号提供者上，刚好2号提供者宕机了，那么它会自动创建一个2号虚拟主机，请求仍然落到2号上，只不过2号虚拟主机会将任务平摊到其他提供者主机上。默认会创建160个虚拟主机。

### (2) 指定负载均衡算法

#### A、 消费者端指定

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0y0kqz3isj313i0aotek.jpg)

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0y0l3mx40j31380eo11h.jpg)

#### B、 提供者端指定

​	在提供者端指定负载均衡算法要求所有该服务的提供者必须指定相同的负载均衡算法。 所以这种指定方式相对于消费者端指定要麻烦很多。

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0y0nl7crzj313c094q8b.jpg)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <dubbo:application name="02-provider-zk02"/>

    <dubbo:protocol name="dubbo" port="20882"/>

    <!--指定服务注册中心：zk单机-->
    <dubbo:registry address="zookeeper://zkOS:2181"/>

    <bean id="someService" class="com.abc.provider.SomeServiceImpl"/>

    <dubbo:service interface="com.abc.service.SomeService" loadbalance="roundrobin"
                   ref="someService" cluster="Failfast">
        <dubbo:method name="hello" loadbalance="leastactive" retries="3">
            <dubbo:parameter key="hash.arguments" value="0,1,0"/>
            <dubbo:parameter key="hash.nodes" value="320"/>
        </dubbo:method>
        <dubbo:method name="hello2">
        <!-- 指定对第几个参数进行hash运算 -->
            <dubbo:parameter key="hash.arguments" value="0,1,0"/>
            <!-- 虚拟主机数量配置 -->
            <dubbo:parameter key="hash.nodes" value="320"/>
        </dubbo:method>
    </dubbo:service>

</beans>
```

# 3.5 集群容错

​	集群容错指的是,当消费者调用提供者集群时发生异常的处理方案。

## 3.5.1 Dubbo 内置的容错机制

​	Dubbo 内置了 6 种集群容错机制。

### (1) Failover

​	故障转移机制。当消费者调用提供者集群中的某个服务器失败时,其会自动尝试着调用其它服务器。该机制通常用于**读操作**,例如,消费者要通过提供者从 DB 中读取某数据。但重试会带来服务延迟。可以通过如下方式来设置重试次数,注意设置的是重试次数,不含第一次正常调用。

#### A、 提供者端设置

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0y2lf9xi6j312y0d244r.jpg)

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0y2m0vlzgj31360eutg6.jpg)

#### B、 消费者端设置

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0y2nhj8fcj312y0b0af9.jpg)

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0y2ns1k2cj31300cytew.jpg)

### (2) Failfast

​	快速失败机制。消费者端只发起一次调用,若失败则立即报错。通常用于**非幂等性的写操作**,比如新增记录。 

### (3) Failsafe

​	失败安全机制。当消费者调用提供者出现异常时,直接忽略本次消费操作。该机制通常用于**执行相对不太重要的服务**,例如,写入审计日志等操作。

### (4) Failback

​	失败自动恢复机制。消费者端调用提供者失败后,Dubbo 会记录下该失败请求,然后定时自动重新发送该请求。该机制通常用于**实时性要求不太高的服务**,例如消息通知操作。 

### (5) Forking

​	并行机制。消费者对于同一服务并行调用多个提供者服务器,只要一个成功即调用结束并返回结果。通常用于**实时性要求较高的读操作**,但其会浪费较多服务器资源。 

### (6) Broadcast

​	广播机制。广播调用所有提供者,逐个调用,任意一台报错则报错。通常用于通知所有提供者更新缓存或日志等本地资源信息。

## 3.5.2 指定集群容错机制

​	**Dubbo 默认的容错机制是故障转移机制 Failover**,可以通过如下设置修改容错机制。

### (1) 提供者端

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0y2vhp119j312g0b8afc.jpg)

### (2) 消费者端

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0y2w6bqqlj312g09wtdm.jpg)

# 3.6 服务降级

## 3.6.1 服务降级基础(面试题)

### (1) 什么是服务降级

> ​	服务降级,当服务器压力剧增的情况下,根据当前业务情况及流量对一些服务有策略的降低服务级别,以释放服务器资源,保证核心任务的正常运行。例如,双 11 时 0 点-2 点期间淘宝用户不能修改收货地址,不能查看历史订单,就是典型的服务降级。

### (2) 服务降级方式

​	能够实现服务降级方式很多,常见的有如下几种情况:

> * 部分服务暂停
> * 部分服务延迟
> * 全部服务暂停 
> * 随机拒绝服务 

### (3) 整个系统的服务降级埋点

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0y2zm5t0bj31360h4q99.jpg)

### (4) 服务降级与 Mock 机制

​	Dubbo 的服务降级采用的是 mock 机制(mock,虚假的,模仿的)。其具有两种降级处理方式：**Mock Null 降级处理**与 **Class Mock 降级处理**。

## 3.6.2 Mock Null 服务降级处理 06-consumer-downgrade

​	本例要测试的是当服务提供者提供服务失败时消费者的服务降级处理,不启动服务提供者就相当于服务提供失败,所以这里就无需定义提供者了。 

### (1) 创建消费者工程

​	直接复制 `02-consumer-zk` 工程,并命名为 `06-consumer-downgrade`。

### (2) 定义接口

​	Dubbo 对于有返回值与无返回值的业务方法的服务降级处理方式是不同的,所以我们这里就不再使用 00-api 工挰中的接口了,而是在当前工程中直接定义一个接口 UserService,该接口中包含两个方法,一个有返回值,一个没有。

```java
///06-consumer-downgrade2/src/main/java/com/abc/service/UserService.java
public interface UserService {
    String getUsernameById(int id);
    void addUser(String username);
}
```

### (3) 修改 pom 文件

​	由于这里不再需要 00-api 工程了,所以在 pom 文件中将对 00-api 工程的依赖删除即可。

### (4) 修改 spring-consumer.xml

​	`mock="return null"`,指定这里使用的是 `Mock Null` 服务降级处理方式。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    
    <dubbo:application name="06-consumer-downgrade"/>

    <dubbo:registry address="zookeeper://zkOS:2181" />

    <dubbo:reference id="userService" check="false" mock="return null"
                     interface="com.abc.service.UserService"/>

</beans>
```

### (5) 修改消费者启动类

​	`Mock Null` 服务降级处理方式,对于有返回值的方法降级的结果是 null,而对于没有返回值的方法降级后的结果是无任何显示,因为其根本就没有返回值,当然,也不会报错。 

```java
public class ConsumerRun {
    public static void main(String[] args) {
        ApplicationContext ac = new ClassPathXmlApplicationContext("spring-consumer.xml");
        UserService service = (UserService) ac.getBean("userService");

        // 有返回值的方法降级结果为null
        String usernameById = service.getUsernameById(3);
        System.out.println(usernameById);

        // 无返回值的方法降级结果是无任何显示
        service.addUser("China");
    }
}

```

## 3.6.3 Class Mock 服务降级处理 06-consumer-downgrade2

​	`Mock Null` 降级处理不好的地方是,没有返回值的方法没有任何返回结果,而具有返回值的方法也只会返回 null,不会返回其它结果。Class Mock 服务降级方式解决了这两个问题。 

### (1) 创建消费者工程

​	本例直接在前面的例子基础上进行修改。

### (2) 定义 Mock Class

​	在**==业务接口所在的包中==**,本例为 com.abc.service 包,定义一个类,该类的命名需要满足以下规则:业务接口简单类名 + Mock。本例应为 UserServiceMock。该类要实现业务接口,而方法的内容即为服务降级后要要执行的代码逻辑。

```java
///06-consumer-downgrade2/src/main/java/com/abc/service/UserServiceMock.java
public class UserServiceMock implements UserService {

    @Override
    public String getUsernameById(int id) {
        return "没有该用户：" + id;
    }

    @Override
    public void addUser(String username) {
        System.out.println("添加该用户失败：" + username);
    }
}
```

### (3) 修改 spring-consumer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    
    <dubbo:application name="06-consumer-downgrade2"/>

    <dubbo:registry address="zookeeper://zkOS:2181" />

    <dubbo:reference id="userService" check="false" mock="true"
                     interface="com.abc.service.UserService"/>

</beans>
```

# 3.7 服务限流

​	Dubbo 中能够实现服务限流的方式较多,可以划分为两类:**直接限流**与**间接限流**。

## 3.7.1 直接限流

​	所谓直接限流是指,通过**对连接的数量直接限制**来达到限流的目的。超过限制则会让消费者等待,直到等待超时,或获取到相应服务。

### (1) executes 限流

​	该属性==**仅能设置在提供者端**==。可以设置为**接口级别**,也可以设置为**方法级别**。

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0y3fmgo39j31320agwjr.jpg)

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0y3fuqo4tj312k0cin41.jpg)

### (2) accepts 限流

​	该属性==仅可设置在提供者端的`<dubbo:provider/>`与`<dubbo:protocol/>`标签中==,用于对指定协议连接数的限制。

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0y3hm2xb0j312k07678h.jpg)

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0y3hvruqmj312q07awik.jpg)

### (3) actives 限流

​	该限流方式与前两种不同的是,其可以设置在提供者端,也可以设置在消费者端。

#### A、 提供者端限流

​	actives 属性设置在提供者端的功能与 executes 的相同。可以设置为接口级别,也可以设置为方法级别。 

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0y3k3yqgjj312k08wn2g.jpg)

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0y3kcvb4xj312u0cm0zk.jpg)

#### B、 消费者端限流

​	可以设置为接口级别,也可以设置为方法级别。

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0y3l7vvt2j312m08gtdf.jpg)

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0y3li696yj312u0ay0yg.jpg)

### (4) connections 限流

​	该限流方式与 actives 的效果基本相同,其同样可以设置在提供者端,也可以设置在消费者端。 

#### A、 提供者端限流

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0y3mfsf50j312o09oq8c.jpg)

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0y3mqluguj312k0c6dmb.jpg)

#### B、 消费者端限流

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0y3nefyoxj312808etdg.jpg)

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0y3nm5gb2j312k0asteq.jpg)

## 3.7.2 间接限流

​	所谓间接限流是指,通过一些非连接数量设置的间接手段来达到限流的目的。

### (1) 延迟连接

​	延迟连接用于减少==长连接数==。只有当消费者端的调用发起时,才创建长连接,所以其是设置在消费者端的。由于其仅作用于长连接,所以其仅对于 Dubbo 服务暴露协议生效。 

​	注意：该属性仅可设置在消费者端的`<dubbo:reference/>`与`<dubbo:consumer/>`标签 中,且也==不能设置为方法级别==。

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0y3qbcyw5j312k08adkf.jpg)

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0y3que6xzj312w06qgoz.jpg)

### (2) 粘连连接

​	粘连连接限制的不是流量,而是流向。

​	所谓粘连连接是指,尽可能让客户端总是向同一提供者发起调用,除非该提供者挂了, 再连另一台。粘滞连接将自动开启延迟连接,以减少长连接数。 

​	粘连连接==仅能设置在消费者端==,其可以设置为接口级别,也可以设置为方法级别。 

 ![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0y3sikgh2j312u08kjwd.jpg)

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0y3ssih8aj312y0bygrv.jpg)

### (3) 负载均衡

​	将负载均衡策略设置为 leastactive 的方式,可以使消费者端调用并发数最少的提供者,以达到限流的目的。所以,该方式限制的也不是流量,而是流向。当然,可以设置在消费者端,亦可设置在提供者端;可以设置在接口级别,亦可设置在方法级别。

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0y3twvpy3j312s07q43e.jpg)

# 3.8 声明式缓存

​	为了进一步提高消费者对用户的响应速度,减轻提供者的压力,Dubbo 提供了**基于结果的声明式缓存**。该缓存是**基于消费者端**的,所以使用很简单,只需修改消费者配置文件,与提供者无关。 

​	该缓存是缓存在消费者端内存中的,一旦缓存创建,即使提供者宕机也不会影响消费者 端的缓存。

## 3.8.1 缓存设置 07-consumer-cache

### (1) 创建工程

​	直接复制 `02-consumer-zk` 工程,并命名为 `07-consumer-cache`。 

### (2) 修改消费者配置文件

​	仅需在`<dubbo:reference/>`中添加 `cache="true"`属性即可:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    
    <dubbo:application name="07-consumer-cache"/>

    <dubbo:registry address="zookeeper://zkOS:2181" />

    <dubbo:reference id="someService"  cache="true"
                     interface="com.abc.service.SomeService"/>

</beans>
```

### (3) 修改 RunConsumer 类

```java
///07-consumer-cache/src/main/java/com/abc/consumer/ConsumerRun.java
public class ConsumerRun {
    public static void main(String[] args) {
        ApplicationContext ac = new ClassPathXmlApplicationContext("spring-consumer.xml");
        SomeService service = (SomeService) ac.getBean("someService");

        System.out.println(service.hello("Tom"));
        System.out.println(service.hello("Jerry"));
        System.out.println(service.hello("Tom"));
        System.out.println(service.hello("Jerry"));
    }
}
```

### (4) 方法级别的结果缓存

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0ywpdy1a9j312m0c2q8c.jpg)

## 3.8.2 默认缓存 1000 个结果

​	声明式缓存中可以缓存多少个结果呢?默认可以缓存 1000 个结果。若超出 1000,将采 用“最近最少使用策略”,即 LRU 策略来删除缓存,以保证最热的数据被缓存。注意,该删 除缓存的策略不能修改。 

​	直接在 `07-consumer-cache` 工程中创建 `ConsumerRun2` 类即可。 

```java
public class ConsumerRun2 {
    public static void main(String[] args) {
        ApplicationContext ac =
                new ClassPathXmlApplicationContext("spring-consumer.xml");
        SomeService service = (SomeService) ac.getBean("someService");

        // 1000次不同的消费结果，将占满1000个缓存空间
        for (int i=1; i<=1000; i++) {
            service.hello("i==" + i);
        }

        // 第1001次不同的消费结果，会将第1个缓存内容挤出去
        System.out.println(service.hello("Tom"));

        // 本次消费会从调用提供者，说明原来的第1个缓存的确被挤出去了
        // 本次消费结果会将原来("i==2")的缓存结果挤出去
        System.out.println(service.hello("i==1"));

        // 本次消费会直接从缓存中获取
        System.out.println(service.hello("i==3"));
    }
}
```

# 3.9 多注册中心

## 3.9.1 创建提供者 08-provider-registers

### (1) 创建工程

​	直接复制 `05-provider-group` 工程,并命名为 `08-provider-registers`。

### (2) 修改配置文件

​	同一个服务注册到不同的中心,使用逗号进行分隔。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <dubbo:application name="08-provider-registers"/>

    <!--声明注册中心-->
    <dubbo:registry id="bjCenter" address="zookeeper://bjZK:2181"/>
    <dubbo:registry id="shCenter" address="zookeeper://shZK:2181"/>
    <dubbo:registry id="gzCenter" address="zookeeper://gzZK:2181"/>
    <dubbo:registry id="cqCenter" address="zookeeper://cqZK:2181"/>

    <!--注册Service实现类-->
    <bean id="weixinService" class="com.abc.provider.WeixinServiceImpl"/>
    <bean id="zhifubaoService" class="com.abc.provider.ZhifubaoServiceImpl"/>

    <!--暴露服务：同一个服务注册到不同的中心；不同的服务注册到不同的中心-->
    <dubbo:service interface="com.abc.service.SomeService"
                   ref="weixinService" group="pay.weixin" register="bjCenter, shCenter"/>
    <dubbo:service interface="com.abc.service.SomeService"
                   ref="zhifubaoService" group="pay.zhifubao" register="gzCenter, cqCenter"/>
</beans>
```

## 3.9.2 创建消费者 08-consumer-registers

### (1) 创建工程

​	直接复制 `05-consumer-group` 工程,并命名为 `08- consumer-registers`。

### (2) 修改配置文件

​	对于消费者工程,用到哪个注册中心了,就声明哪个注册中心,无需将全部注册中心进行声明。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    
    <dubbo:application name="08-consumer-registers"/>

    <!--声明注册中心-->
    <dubbo:registry id="bjCenter" address="zookeeper://bjZK:2181"/>
    <dubbo:registry id="gzCenter" address="zookeeper://gzZK:2181"/>

    <!--指定调用bjCenter注册中心微信服务-->
    <dubbo:reference id="weixin"  group="pay.weixin" registry="bjCenter"
                     interface="com.abc.service.SomeService"/>

    <!--指定调用gzCenter与cqCenter注册中心支付宝服务-->
    <dubbo:reference id="gzZhifubao"  group="pay.zhifubao" registry="gzCenter"
                     interface="com.abc.service.SomeService"/>
    <dubbo:reference id="cqZhifubao"  group="pay.zhifubao" registry="cqCenter"
                     interface="com.abc.service.SomeService"/>

</beans>
```

# 3.10 单功能注册中心

## 3.10.1 仅订阅应用场景

​	为了方便,我们会在开发和测试时共用同一个注册中心,但可能会出现正在开发的服务注册到注册中心后,原来运行没有问题的服务出现了问题。那也就是说,该服务在开发完毕之前不能注册到注册中心。因为之前的连接注册中心的方式是,提供者只要连接上了注册中心,其就会自动注册到中心,所以该正在开发的服务根本就不能连接注册中心,否则其就自动注册了。但该正在开发的服务,其运行还需要调用注册中心中的其它服务,即该服务还必须要连接上注册中心,否则无法调用其它服务。这就出现了矛盾。 

​	也就是说,我们现在需要的场景是:提供者可以连接上注册中心,并消费其中的服务, 但还不会注册到中心。此时,可以使用“仅订阅”模式。

## 3.10.2 仅注册应用场景

​	仅注册是指,提供者仅仅作为提供者注册到了注册中心,但其并不能作为消费者去消费当前注册中心的其它服务。

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0yzm9yumsj30sw0kwk0l.jpg)

​	首先要清楚一点,注册到注册中心的服务不一定都是纯粹的提供者,它们有可能同时也是消费者,它们还需要调用其它服务。 

​	现在要构建一个场景:B 是注册中心 A 的镜像,但还在建设中,A 中的某些服务在 B 中暂时还没有,例如服务 s 在 A 中存在,但在 B 中不存在,而服务 s 暂时还不能注册到中心 B (例如注册服务 s 需要很长时间)。那些陆续在两个注册中心注册的其它提供者很多又都依赖于服务 s。若直接将这些提供者注册到 A 与 B 注册中心,则 B 中心中的这些服务运行可能就会出问题。因为这些服务所依赖的服务 s 在 B 中心没有。此时出现矛盾:镜像注册中心 B 要求在 A 中注册的服务,也必须要在 B 中注册。但在 B 中注册的这些服务又会由于缺少服 务 s 而运行出错。 

​	也就是说,我们现在需要的场景是:注册到 B 中的服务,不订阅(消费、依赖)B 中的服务。  

## 3.10.3 实现 09-provider-oneway

### (1) 创建工程

​	直接复制 `08-provider-registers` 工程,并命名为 `09-provider-oneway`。

### (2) 修改配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <dubbo:application name="09-provider-oneway"/>

    <!--声明注册中心：仅订阅-->
    <dubbo:registry id="bjCenter" address="zookeeper://bjZK:2181" register="false"/>
    <!--声明注册中心：仅注册-->
    <dubbo:registry id="gzCenter" address="zookeeper://gzZK:2181" subscribe="false"/>

    <!--注册Service实现类-->
    <bean id="weixinService" class="com.abc.provider.WeixinServiceImpl"/>
    <bean id="zhifubaoService" class="com.abc.provider.ZhifubaoServiceImpl"/>

    <!--暴露服务-->
    <dubbo:service interface="com.abc.service.SomeService"
                   ref="weixinService" group="pay.weixin" register="bjCenter"/>
    <dubbo:service interface="com.abc.service.SomeService"
                   ref="zhifubaoService" group="pay.zhifubao" register="gzCenter"/>
</beans>
```

# 3.11 服务暴露延迟

​	如果我们的服务启动过程需要 warmup 事件,就可以使用 delay 进行服务延迟暴露。只需在服务提供者的`<dubbo:service/>`标签中添加 `delay` 属性。其值可以有三类: 

*   若为正数,则单位为毫秒,表示在提供者对象创建完毕后的指定时间后再发布服务; 

*   默认值为 0,表示当前提供者创建完毕后马上暴露服务。 

*   若值为-1,则表示在 Spring 容器初始化完毕后再暴露服务。

  ![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0yzvuwwtkj30zc0agdj9.jpg)

  ![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0yzw6toj1j30zs09yjup.jpg)

# 3.12 对提供者的异步调用

## 3.12.1 创建提供者 10-provider-async

 