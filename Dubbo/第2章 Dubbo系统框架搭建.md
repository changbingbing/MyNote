# 	2.1 第一个 Dubbo 程序(直连式)

## 2.1.1 创建业务接口工程 00-api

​	业务接口名即服务名称。无论是服务提供者向服务注册中心注册服务,还是服务消费者从注册中心索取服务,都是通过接口名称进行注册与查找的。即,提供者与消费者都依赖于业务接口。所以,一般情况下,会将业务接口专门定义为一个工程,让提供者与消费者依赖。

### (1) 创建 Maven 的 Java 工程

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0xm2uqq60j30ju0gyq5r.jpg)



### (2) 创建业务接口

```java
/**
 * 业务接口
 */
///00-api/src/main/java/com/abc/service/SomeService.java
public interface SomeService {
    String hello(String name);
}
```

### (3) 修改 pom 文件

​	这个POM中无任何依赖

```xml
<modelVersion>4.0.0</modelVersion>

    <groupId>com.abc</groupId>
    <artifactId>00-api</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>
```

### (4) 对工程执行 install

​	将工程 install 到本地仓库,以备其它工程依赖。

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0xm6yq30nj30ge0mctc3.jpg)

## 2.1.2 创建提供者(自建 Spring 容器)01-provider

### (1) 创建工程

​	创建一个 Maven 的 Java 工程,并命名为 `01-provider`。

### (2) 在 pom 中导入依赖

​	依赖导入，主要包括三类依赖：

```xml
<properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <!-- 自定义版本号 -->
        <spring-version>4.3.16.RELEASE</spring-version>
    </properties>

    <dependencies>
        <!--业务接口工程依赖-->
        <dependency>
            <groupId>com.abc</groupId>
            <artifactId>00-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <!-- dubbo依赖 -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.7.0</version>
        </dependency>

        <!-- Spring依赖 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-expression</artifactId>
            <version>${spring-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>${spring-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>${spring-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>${spring-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring-version}</version>
        </dependency>
        <!-- commons-logging依赖 -->
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
        </dependency>

    </dependencies>
```

### (3) 定义接口实现类

```java
///01-provider/src/main/java/com/abc/provider/SomeServiceImpl.java
public class SomeServiceImpl implements SomeService {
    @Override
    public String hello(String name) {
        System.out.println(name + "，我是提供者");
        return "Hello Dubbo World! " + name;
    }

}
```

### (4) 定义 spring-provider 配置文件

​	在 下定义 spring-provider.xml 配置文件:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!--指定当前工程在Monitor中显示的名称，一般与工程名相同-->
    <dubbo:application name="01-provider"/>

    <!--指定服务注册中心：不指定注册中心，即注册中心不可用！后面计划采用直连方式-->
    <dubbo:registry address="N/A"/>

    <!--注册服务执行对象（这里用扫描器也可以）-->
    <bean id="someService" class="com.abc.provider.SomeServiceImpl"/>

    <!--服务暴露，即服务注册-->
    <dubbo:service interface="com.abc.service.SomeService"
                   ref="someService"/>
</beans>
```

### (5) 启动类

```java
///01-provider/src/main/java/com/abc/run/ProviderRun.java
public class ProviderRun {
    public static void main(String[] args) throws IOException {
        // 创建Spring容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("spring-provider.xml");
        // 启动Spring容器
        ((ClassPathXmlApplicationContext) ac).start();
        // 使主线程阻塞
        System.in.read();
    }
}
```

## 2.1.3 创建提供者(Main 启动) 01-provider2

​	Dubbo 提供了一个 `Main.main()`方法可以直接启动 Provider。其要求 Spring 配置文件必须要放到类路径下的 `META-INF/spring` 目录中,Spring 配置文件名称无所谓。 

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0xmnnihs8j30gy06qq47.jpg)

### (1) 工程创建

​	复制前面的 Provider 工程,并修改其工程名 `01-provider2`

### (2) 创建目录并移动配置文件

​	在 resources 目录中创建 `META-INF/spring` 目录,并将 `spring-provider.xml` 配置文件拖入其中 

### (3) 修改启动类

```java
import org.apache.dubbo.container.Main;
import java.io.IOException;

public class ProviderRun {
    public static void main(String[] args) throws IOException {
        Main.main(args);
    }
}
```

### (4) 启动运行

​	当在控制台看到如下内容时说明启动成功。

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0xmr07ymqj312w08en0z.jpg)

## 2.1.4 创建消费者 01-consumer

### (1) 创建工程

​	创建一个 Maven 的 Java 工程,并命名为 `01-consumer`。

### (2) 在 pom 中导入依赖

​	该工程的依赖与 Provider 中的完全相同,直接复制来就可以。

### (3) 定义 spring-consumer 配置文件

​	在 src/main/resources 下定义 spring-consumer.xml 配置文件。

```xml
<!--指定当前工程在Monitor中显示的名称，一般与工程名相同-->
    <dubbo:application name="01-consumer"/>

    <!--指定服务注册中心：不指定注册中心-->
    <dubbo:registry address="N/A"/>

    <!--订阅服务：采用直连式连接消费者-->
    <dubbo:reference id="someService"
                     interface="com.abc.service.SomeService"
                     url="dubbo://localhost:20880"/>
```

### (4) 定义消费者类

```java
public class ConsumerRun {
    public static void main(String[] args) {
        ApplicationContext ac = new ClassPathXmlApplicationContext("spring-consumer.xml");
        SomeService service = (SomeService) ac.getBean("someService");
        String hello = service.hello("China");
        System.out.println(hello);
    }
}
```

# 2.2 Zookeeper 注册中心

## 2.2.1 创建提供者 02-provider-zk

### (1) 导入依赖

​	复制前面的提供者工程 `01-provider`,并更名为 `02-provider-zk`。修改 pom 文件,并在其中导入 Zookeeper 客户端依赖 curator。

```xml
<!-- zk客户端依赖：curator -->
<!-- 注意两点：
	一、这里依赖的curator2.13.0是2.x版本中的最高的版本，3.x版本的话，api就不支持了
	二、Dubbo2.6.x版本只需要依赖curator-framework即可，而到了2.7.x版本后还需要依赖curator-recipes
-->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>2.13.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>2.13.0</version>
        </dependency>
```

### (2) 修改 spring 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <dubbo:application name="02-provider-zk"/>

    <!--指定服务注册中心：zk单机的两种写法-->
    <dubbo:registry address="zookeeper://zkOS:2181"/>
    <!--<dubbo:registry protocol="zookeeper" address="zkOS:2181"/>-->

    <!--指定服务注册中心：zk集群的两种写法-->
    <!--<dubbo:registry address="zookeeper://zkOS1:2181?backup=zkOS2:2181,zkOS3:2181,zkOS4:2181"/>-->
    <!--<dubbo:registry protocol="zookeeper" address="zkOS1:2181,zkOS2:2181,zkOS3:2181,zkOS4:2181"/>-->

    <bean id="someService" class="com.abc.provider.SomeServiceImpl"/>

    <dubbo:service interface="com.abc.service.SomeService"
                   ref="someService" />

</beans>
```

## 2.2.2 创建消费者 02-consumer-zk

### (1) 导入依赖

​	复制前面的消费者工程 `01-consumer`,并更名为 `02-consumer-zk`。修改 pom 文件,并在其中导入 Zookeeper 客户端 curator 依赖。

```xml
<!-- zk客户端依赖：curator -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>2.13.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>2.13.0</version>
        </dependency>
```

### (2) 修改 Spring 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    
    <dubbo:application name="02-consumer-zk"/>

    <!--指定服务注册中心：zk单机-->
    <dubbo:registry address="zookeeper://zkOS:2181" />
    <!--<dubbo:registry protocol="zookeeper" address="zkOS:2181"/>-->

    <!--指定服务注册中心：zk集群-->
    <!--<dubbo:registry address="zookeeper://zkOS1:2181?backup=zkOS2:2181,zkOS3:2181,zkOS4:2181"/>-->
    <!--<dubbo:registry protocol="zookeeper" address="zkOS1:2181,zkOS2:2181,zkOS3:2181,zkOS4:2181"/>-->

    <dubbo:reference id="someService"  interface="com.abc.service.SomeService"
                     loadbalance="leastactive" retries="2"/>

</beans>
```

# 2.3将 Dubbo 应用到 web 工程

​	前面所有提供者与消费者均是 Java 工程,而在生产环境中,它们都应是 web 工程,Dubbo 如何应用于 Web 工程中呢?

## 2.3.1 创建提供者 03-provider-web

### (1) 创建工程

​	创建 Maven 的 web 工程,然后将 `02-provider-zk` 中的 Service 实现类及 `spring-provider` 配置文件复制到当前工程中。

### (2) 导入依赖

​	这里使用的 Spring 的版本为 4.3.16。需要的依赖🈶️：

> * dubbo2.7.0 版本依赖 
> * zk 客户端 curator 依赖 
> * servlet 与 jsp 依赖 
> * spring 相关依赖
> * spring 需要的 commons-logging 依赖
> * 自定义 00-api 依赖

```xml
<modelVersion>4.0.0</modelVersion>

    <groupId>com.abc</groupId>
    <artifactId>03-provider-web</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <!-- 自定义版本号 -->
        <spring-version>4.3.16.RELEASE</spring-version>
    </properties>

    <dependencies>
        <!-- zk客户端依赖：curator -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>2.13.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>2.13.0</version>
        </dependency>

        <!-- dubbo依赖 -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.7.0</version>
        </dependency>

        <!-- Spring依赖 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-expression</artifactId>
            <version>${spring-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>${spring-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>${spring-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>${spring-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>${spring-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring-version}</version>
        </dependency>

        <!-- commons-logging依赖 -->
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
        </dependency>

        <!-- Servlet依赖 -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
        </dependency>

        <!-- JSP依赖 -->
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>javax.servlet.jsp-api</artifactId>
            <version>2.2.1</version>
            <scope>provided</scope>
        </dependency>

        <!--业务接口工程依赖-->
        <dependency>
            <groupId>com.abc</groupId>
            <artifactId>00-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

    </dependencies>

    <build>
        <finalName>03-provider-web</finalName>
        <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
            <plugins>
                <plugin>
                    <artifactId>maven-clean-plugin</artifactId>
                    <version>3.0.0</version>
                </plugin>
                <!-- see http://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_war_packaging -->
                <plugin>
                    <artifactId>maven-resources-plugin</artifactId>
                    <version>3.0.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.7.0</version>
                </plugin>
                <plugin>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <version>2.20.1</version>
                </plugin>
                <plugin>
                    <artifactId>maven-war-plugin</artifactId>
                    <version>3.2.0</version>
                </plugin>
                <plugin>
                    <artifactId>maven-install-plugin</artifactId>
                    <version>2.5.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-deploy-plugin</artifactId>
                    <version>2.8.2</version>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
```

### (3) 定义 web.xml 

```xml
<!--注册Spring配置文件-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-*.xml</param-value>
    </context-param>

    <!--注册ServletContext监听器-->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
```

### (4) 修改 spring-provider.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!--指定当前工程在Monitor中显示的名称，一般与工程名相同-->
    <dubbo:application name="03-provider-web"/>

    <!--指定服务注册中心：zk单机-->
    <dubbo:registry address="zookeeper://zkOS:2181"/>

    <!--注册服务执行对象-->
    <bean id="someService" class="com.abc.provider.SomeServiceImpl"/>

    <!--服务暴露-->
    <dubbo:service interface="com.abc.service.SomeService"
                   ref="someService" />

</beans>
```

## 2.3.2 创建消费者 03-consumer-web

### (1) 创建工程

​	创建 Maven 的 web 工程,然后将 `02-consumer-zk` 中的 `spring-provider` 配置文件复制到当前工程中。

### (2) 导入依赖

​	与提供者工程中的依赖相同。

### (3) 定义处理器

```java
///03-consumer-web/src/main/java/com/abc/controller/SomeController.java
@Controller
public class SomeController {
    @Autowired
    private SomeService service;

    @RequestMapping("/some.do")
    public String someHandle() {
        String result = service.hello("China");
        System.out.println("消费者端接收到 = " +  result);
        return "/welcome.jsp";
    }
}
```

### (4) 定义欢迎页面

​	在 src/main/webapp 目录下定义欢迎页面。

```html
<!-- /03-consumer-web/src/main/webapp/welcome.jsp -->
<html>
<body>
<h2>welcome you!</h2>
</body>
</html>
```

### (5) 定义 web.xml

```xml
<!-- Dubbo2.6.4中需要该配置，否则找不到配置文件；2.7.x版本则不需要 -->
    <!--<context-param>-->
        <!--<param-name>contextConfigLocation</param-name>-->
        <!--<param-value>classpath:spring-*.xml</param-value>-->
    <!--</context-param>-->

<!--字符编码过滤器-->
    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!--注册中央调度器-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-*.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <!--不能写/*，不建议写/，建议扩展名方式-->
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
```

### (6) 修改 spring-consumer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns:mvc="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    
    <dubbo:application name="03-consumer-web"/>

    <!--指定服务注册中心：zk单机-->
    <dubbo:registry address="zookeeper://zkOS:2181"/>

    <!--订阅服务-->
    <dubbo:reference id="someService" check="false"
                     interface="com.abc.service.SomeService"/>

    <!--注册处理器-->
    <mvc:component-scan base-package="com.abc.controller"/>
</beans>
```

### (7) 关闭服务检查

​	默认情况下,若服务消费者等于服务提供者启动,则消费者端会报错。因为默认情况下消费者会在启动时查检其要消费的服务的提供者是否已经启动,若未启动则抛出异常。可以在消费者端的 spring 配置文件中添加 `check="false"`属性,则可关闭服务检查功能。

```xml
<!--订阅服务-->
    <dubbo:reference id="someService" check="false"
                     interface="com.abc.service.SomeService"/>
```

​	⚠️⚠️⚠️只要注意启动顺序,该属性看似可以不使用。但在循环消费场景下是必须要使用的。即A 消费 B 服务,B 消费 C 服务,而 C 消费 A 服务。这是典型的循环消费。在该场景下必须至少有一方要关闭服务检查功能,否则将无法启动任何一方。

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0xp35wkr6j30oa0k4adg.jpg)

## 2.3.3 部署运行

​	为了方便部署测试,这里将提供者与消费者部署到一个 Tomcat 中。

### (1) 设置 Tomcat

​	对于 Tomcat 的设置,需要注意以下两点:

1. 修改 Tomcat 默认端口号:
   由于 Dubbo 管控台端口号为 8080,所以这里将 Tomcat 默认端口号修改为 8081。
   ![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0xp4z5dblj30zs0u0aj4.jpg)
2. 两个应用两个 context
   ![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0xp5oelnwj312m08sq5c.jpg)
   ![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0xp67skrbj312e09276r.jpg)

### (2) 运行

1. 启动 Tomcat 后,在浏览器提交如下请求,可以看到 welcome.jsp 页面。
   ![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0xp75hg92j30zu0cawia.jpg)
2. 控制台中可以看到如下结果。
   ![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0xp80rqmnj31260ea0xg.jpg)

# 2.4Dubbo 管理控制台

​	Dubbo 进入孵化器之前是没有 Dubbo 管理控制台的,Dubbo 的服务治理与监控由两个平台完成:服务治理平台与服务监控平台。进入孵化器后出现了前面的平台不能使用,而后面的平台尚未开发的境况。

​	2019 年初,官方发布了 Dubbo 管理控制台 0.1 版本。结构上采取了前后端分离的方式, 前端使用 Vue 和 Vuetify 分别作为 Javascript 框架和 UI 框架,后端采用 Spring Boot 框架。 

## 2.4.1 下载

​	Dubbo 管理控制台的下载地址为:https://github.com/apache/incubator-dubbo-ops

## 2.4.2 配置

​	在下载的 zip 文件的解压目录的 `dubbo-admin-server\src\main\resources` 下,修改配置文件`application.properties`。主要就是修改注册中心、配置中心,与元数据中心的 zk 地址。 

​	这是一个 springboot 工程,默认端口号为 8080,若要修改端口号,则在配置文件中增 加形如 `server.port=8888` 的配置。 

## 2.4.3 打包

1. 在命令行窗口中进入到解压目录根目录,执行打包命令。
   ![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0xpd7he9mj30tc084tar.jpg)

2. 当看到以下提示时表示打包成功。
   ![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0xpdy0uhwj312a0t21ay.jpg)

3. 打包结束后,进入到解压目录下的 `dubbo-admin-distribution` 目录下的 target 目录。
   ![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0xpezwt8sj30mg0iqdiw.jpg)

   该目录下有个 dubbo-admin-0.1.jar 文件。该 Jar 包文件即为 Dubbo 管理控制台的运行文件,可以将其放到任意目录下运行。
   ![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0xpg626bcj30qw0gugp9.jpg)

## 2.4.4 运行

### (1) 启动 zk

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0xpguxcmgj30x60bkq8v.jpg)

### (2) 启动管控台

​	将 dubbo-admin-0.1.jar 文件存放到任意目录下,例如 D 盘根目录下,直接运行.

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0xphssclpj30ks07eabq.jpg)

### (3) 访问

​	在浏览器地址栏中输入 http://localhost:8080 ,即可看到 Dubbo 管理控制台界面。

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0xpido4jsj30w10u044v.jpg)

