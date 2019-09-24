# 自定义异常页面

​	对于 404、405、500 等异常状态,服务器会给出默认的异常页面,而这些异常页面一般 都是英文的,且非常不友好。我们可以通过简单的方式使用自定义异常页面,并将默认状态码页面进行替换。 

​	直接在前面程序上修改即可,无需创建新的工程。 

## 定义目录

在 src/main/resources 目录下再定义新的目录 ==public/error==,**必须**是这个目录名称。

## 定义异常页面

在 error 目录中定义异常页面。这些异常页面的名称必须为相应的状态码,扩展名为 html。

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d26h1hnlj30ke0hwtc0.jpg)

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d26yx12zj30n20l279c.jpg)

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6d277kjjgj30o60l878w.jpg)

## 修改处理器模拟500错误

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d283d9i8j30sq0gsn1z.jpg)

## 访问效果

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6d28cogb2j30t00d8n0n.jpg)

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d28t1x1dj30r20c6juo.jpg)

# 单元测试

## 定义工程

直接在前面工程中修改即可。

## 定义Service接口

```java
public interface SomeService {
	void doSome();
}
```

## 定义Service两个实现类

注意⚠️：实现类上要添加```@Service``` 注解,以交给 Spring 容器来管理。

```java
@Service
public class OtherServiceImpl implements SomeService {

	@Override
	public void doSome() {
		// TODO Auto-generated method stub
		System.out.println("OtherServiceImpl的doSome()");
	}

}
```

```java
@Service
public class SomeServiceImpl implements SomeService {

	@Override
	public void doSome() {
		// TODO Auto-generated method stub
		System.out.println("SomeServiceImpl的doSome()");
	}

}
```

## 修改测试类

打开 src/test/java 中的测试类,在其中直接添加测试方法即可。

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class ApplicationTests {
	
	//若接口只有一个实现类，则可以使用byType方式自动注入；但现在存在两个实现类，只能使用byName方式自动注入
	@Autowired
	@Qualifier("someServiceImpl")
	private SomeService someService;
	
	@Autowired
	@Qualifier("otherServiceImpl")
	private SomeService otherService;
	
	@Test
	public void someTest() {
		this.someService.doSome();
		this.otherService.doSome();
	}
}
```

# 多环境选择

## 什么是多环境选择

以下两种场景下需要进行“多环境选择”。

### 相同代码运行在不同环境

​	在开发应用时,通常同一套程序会被运行在多个不同的环境,例如,开发、测试、生产 环境等。每个环境的数据库地址、服务器端口号等配置都会不同。若在不同环境下运行时将 配置文件修改为不同内容,那么,这种做法不仅非常繁琐,而且很容易发生错误。 

​	此时就需要定义出不同的配置信息,在不同的环境中选择不同的配置。 

### 不同环境执行不同实现类

​	在开发应用时,有时不同的环境,需要运行的接口的实现类也是不同的。例如,若要开 发一个具有短信发送功能的应用,开发环境中要执行的 send()方法仅需调用短信模拟器即可, 而生产环境中要执行的 send()则需要调用短信运营商所 供的短信发送接口。 

​	此时就需要开发两个相关接口的实现类去实现 send()方法,然后在不同的环境中自动选 择不同的实现类去执行。 

## 需求

​	下面将实现如下功能:存在开发与生产两种环境,不同环境使用不同配置文件,不同环境调用不同接口实现类。使用不同端口号对不同的配置文件加以区分。 

## 多配置文件实现方式

### 定义工程

复制前面打为 Jar 包的工程,并重命名为 03-multiEnv。

### 定义配置文件

#### 定义多个配置文件

在 src/main/resources 中再定义两个配置文件,分别对应开发环境与生产环境。

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6d2yd0fxij30hi0asdim.jpg)

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d2zc3w8yj30p407otb2.jpg)

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d2zoe5scj30ok076go7.jpg)

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6d2zvl205j30k6078acj.jpg)



#### 说明

> ​	在 Spring Boot 中多环境配置文件名需要满足 application-{profile}.properties 的格式,其 中{profile}为对应的环境标识,例如：
>
> * application-dev.properties:开发环境
> * application-test.properties:测试环境
> * application-prod.properties:生产环境 
>
> ​	至于哪个配置文件会被加载,则需要在 application.properties 文件中通过 spring.profiles.active 属性来设置,其值对应{profile}值。例如,spring.profiles.active=test 就会 加载 application-test.properties 配置文件内容。 
>
> ​	在生产环境下,application.properties 中一般配置通用内容,并设置 spring.profiles.active 属性的值为 dev,即,直接指定要使用的配置文件为开发时的配置文件,而对于其它环境的 选择,一般是通过命令行方式去激活。配置文件 application-{profile}.properties 中则配置各个环境的不同内容。 

### 定义业务代码

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d38w1f8tj30i60dgtcr.jpg)

#### 定义业务代码接口

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6d39dn77wj30ne08w0v0.jpg)

#### 定义两个实现类

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6d3ahbimdj310w0hy43p.jpg)

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6d3arwywyj31040hegqr.jpg)

#### 说明

> 在实现类上添加@Profile 注解,并在注解参数中指定前述配置文件中的{profile}值,用于指定该实现类所适用的环境。

### 定义处理器

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6d3evvryaj30x20kuq9k.jpg)

### idea下运行与访问

#### 运行

​	打开主类在 Idea 中直接运行,即可在控制台看到默认使用的是开发环境,即端口号使用的是 8888,而工程的根路径为/ddd。

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6d3fnm4fvj30zc05wgq8.jpg) 

#### 访问

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6d3g2rvq1j30tk0bitd1.jpg)

​	从页面显示内容可知,其执行的业务接口实现类为 DevelopServiceImpl,即开发阶段应
使用的实现类。

#### 修改配置文件

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d3ghpnj9j30ly07cdi1.jpg)

#### 再运行

​	将上次运行停掉后再次运行主类,即可在控制台看到使用的是生产环境,即端口号使用的是 9999,而工程的根路径为/ppp。

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6d3h2t2ycj30zk06cgpd.jpg)

#### 再访问

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d3hae25wj30u80b2jvu.jpg)

​	从页面显示内容可知,此次执行的业务接口实现类为 ProduceServiceImpl,即生产环境下应使用的实现类

### 在命令行下选择环境

​	将工程打为 Jar 包后,在命令行运行。若要想切换运行环境,必须要修改主配置文件吗? 答案是否定的。只需添加一个命令参数即可动态指定。

#### 在命令行下运行jar包

例如,现在的主配置文件中指定的是 dev 环境。

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d3llnxn2j30ka06kdhz.jpg)

将当前工程打为 Jar 包后,在命令行运行时添加如下参数。

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6d3m68q73j30ze06qjsu.jpg)

此时执行的就是生产环境,调用的就是 ProduceServiceImpl 类。

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d3mixj98j30ss0ak0x0.jpg)

#### 说明

> 在命令行中添加的参数可以是写在配置文件中的任意属性。其原理是命令行设置的属性值的优选级高于配置文件的。

## 单配置文件实现方式

这种实现方式只能使用 application.yml 文件,使用 application.properties 文件好像文件本身就会出错。 

### 定义工程

复制前面的 3-multiEnv 工程,并重命名为 03-multiEnv2。

### 修改配置文件

​	将原有的配置文件全部删除,然后定义 application.yml 文件。需要注意的是,这三部分之间是==由三个减号(-)分隔的,必须是三个==。而这三部分充当着之前的三个配置文件。

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d3qrrm99j30iu10c7ah.jpg)

### 运行与访问

​	运行与访问方式与前面的多配置文件的完全相同。

# 读取自定义配置

## 读取主配置文件中的属性

### 定义工程

复制前面打为 Jar 包的工程,并重命名为 04-customConfig。

### 修改主配置文件

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d3xwtp9zj30ns04ign4.jpg)

### 修改SomeHandler类

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d3zcywg0j30x20l2q9c.jpg)

> 在@Value 注解中通过${ }符号可以读取指定的属性值。

## 读取指定配置文件中的属性

​	一般情况下,主配置文件中存放系统中定义好的属性设置,而自定义属性一般会写入自定义的配置文件中。也就是说,Java 代码除了可以读取主配置文件中的属性外,还可以读取指定配置文件中的属性,可以通过`@PropertySource` 注解加载指定的配置文件。

### 不能自定义yml文件

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6d4aktxn4j31gi0d675j.jpg)

​	spring boot 官网给出说明,`@PropertySource` 注解不能加载 yml 文件。所以其建议自定 义配置文件就使用属性文件。 [官方文档](https://spring.io/)

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d4c1eg3mj31gk0c6wfu.jpg)

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6d4d273kuj31gg0ak770.jpg)

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6d4dkbe5pj31gg0sedlu.jpg)

### 定义工程

复制 04-customConfig 工程,并重命名为 04-customConfig2。

### 修改主配置文件

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d4edxnghj30ec05mjsl.jpg)

### 自定义配置文件

该配置文件为 properties 文件,文件名随意,存放在 src/main/resources 目录中。

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6d4eqqpr8j30ew06cwfk.jpg)

### 修改SomeHandler类

若属性的值存在中文,则需要添加 encoding 属性。

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d4fuxbe2j30zo0gwjxb.jpg)

## 读取对象属性

### 定义工程

复制 04-customConfig2 工程,并重命名为 04-customConfig3。

### 修改自定义配置文件

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d4jsmko3j30ig096mzj.jpg)

此时的 student 称为对象属性。

### 定义配置属性类

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d4m1vroqj30wa0ha0zn.jpg)

说明：

> * @ProertySource用于指定要读取的配置文件
> * @ConfigurationProperties用于指定要读取配置文件中的对象属性,即指定要读取的配置文件属性的前缀
> * @Component表示当前从配置文件读取来的对象,由Spring容器创建
> * 要保证类的属性名要与配置文件中的对象属性名相同 

### 修改SomeHandler类

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d4nsk2s4j30ze0nigtp.jpg)

## 读取List\<String\>属性

### 定义工程

复制 04-customConfig3 工程,并重命名为 04-customConfig4。

### 导入依赖

​	注解@ConfigurationProperties 注解需要以下依赖(该文档笔记Spring Boot版本2.1.1，这里需要导入此依赖，2.1.4版本则不需要)：

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6d569slyjj30zk0a0adv.jpg)

### 修改自定义配置文件

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d56p58faj30lq08gad9.jpg)

### 定义配置属性类

```java
@Data
@Component
@PropertySource("classpath:myapp.properties")
@ConfigurationProperties("country")
public class Country {

	private List<String> cities;
}
```

### 修改SomeHandler类

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d585jgopj30zc0qun6b.jpg)

## 读取List\<Object\>属性

### 定义工程

复制 04-customConfig4 工程,并重命名为 04-customConfig5。

### 修改自定义配置文件

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6d5a51t5cj30m40i0jzd.jpg)

### 定义配置属性类

注意⚠️：Student 类无需任何注解。

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d5ap41l9j30ju0c877p.jpg)

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d5b8d3erj30vo0dijx3.jpg)

### 修改SomeHandler类

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6d5cl0z7kj30zc0q0n5r.jpg)

# Spring Boot下使用JSP页面

## 直接添加JSP文件

### 定义工程

复制第一个工程,并重命名为 06-jsp。

### 创建webapp目录

​	在 src/main 下创建 webapp 目录,用于存放 jsp 文件。这就是一个普通的目录,无需执行 Mark Directory As。

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d6irsz6mj30ms0eawgz.jpg) 

### 创建index.jsp

#### 指定web资源目录

​	在 spring boot 工程中若要创建 jsp 文件,一般是需要在 src/main 下创建 webapp 目录,然后在该目录下创建 jsp 文件。但通过 Alt + Insert 发现没有创建 jsp 文件的选项。此时,需要打开 Project Structrue 窗口,将 webapp 目录指定为 web 资源目录,然后才可以创建 jsp文件。

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6d6ky5d8aj30x80lkq8a.jpg)

指定后便可看到下面的窗口情况。

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d6lg86u4j30zc0lm0xd.jpg)

此时,便可在 webapp 中找到 jsp 的创建选项了。

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d6ltfg0hj30zk0cmtc4.jpg)

#### 创建index页面

​	在 src/main/webapp 下创建一个 html 文件,并命名为 index.html,创建完毕后再将其重命名为 index.jsp。因为 Idea 中是没有 JSP 页面模板的,不能直接创建 JSP 文件。 

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d6mpvqiej30zm0sqdpu.jpg)

### 启动后运行

​	此时启动工程后在浏览器直接访问,发现其并没有显示 index 页面。因为当前工程不能识别 jsp 文件。

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6d6nd4tshj30za0f4wko.jpg)

## 使用物理视图

### 添加jasper依赖

​	在 pom 中添加一个 Tomcat 内嵌的 jsp 引擎 jasper 依赖。jsp 引擎是用于解析 jsp 文件的,即将 jsp 文件解析为 Servlet 是由 jsp 引擎完成的。embed,嵌入。

```xml
<dependency>
			<groupId>org.apache.tomcat.embed</groupId>
			<artifactId>tomcat-embed-jasper</artifactId>
</dependency>
```

### 注册资源目录

在 pom 文件中将 webapp 目录注册为资源目录。

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d6r5kqb7j30zk0s2th8.jpg)

不过,我们一般会添加两个资源目录:

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d7dsw49qj30xv0u0tkh.jpg)

### 创建welcome.jsp

在 webapp 目录下再创建一个子目录 jsp,在其中创建 welcome.jsp 文件。

```jsp
<body>
	name=${pname}<br>
	age=${page}<br>
</body>
```

### 修改SomeHandler类

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6d7g0mbyqj30zc0h0wjm.jpg)

### 访问

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6d7gc57qnj30yi0foaeo.jpg)

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d7gk6ez8j30qq0ee78m.jpg)

## 使用逻辑视图

### 修改主配置文件

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d7hso567j30k60cq0uv.jpg)

### 修改处理器

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d7j4b1hxj30za0gojwm.jpg)

### 访问

执行效果与前面的相同。

# Spring Boot中使用MyBatis

## 需求

完成一个简单的注册功能。

## 定义工程

复制 06-jsp 工程,并重命名为 07-mybatis。

## 修改pom文件

导入三个依赖:mybatis 与 Spring Boot 整合依赖、mysql 驱动依赖与 Druid 数据源依赖。

```xml
<!-- 整合mybatis -->
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>1.3.1</version>
		</dependency>
		<!-- mysql驱动 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
		<!-- druid驱动 -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
			<version>1.1.10</version>
		</dependency>
```

## 修改SomeHandler

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d7tu35tfj310g0nsaiu.jpg)

## 定义Service接口及实现类

### 定义Service接口

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6d7uzxrhjj30q20ag40z.jpg)

### 定义Service实现类

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6d7v82onzj31040jg79v.jpg)

## 定义实体类及DB表

### 定义实体类

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d7w2i83fj30k60dywhz.jpg)

### 定义DB表

在 DB 的 test 数据库中定义 student 表。

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6d7xix0y0j30lw0c8jul.jpg)

## 定义Dao接口

Dao 接口上要添加@Mapper 注解。

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d80u19yrj30rc0b8tb8.jpg)

## 定义映射文件

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6d8165uvtj30z60gajya.jpg)

## 注册资源目录

在 pom 文件中将 dao 目录注册为资源目录。

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6d82h457nj30xx0u07g1.jpg)

## 修改主配置文件

在主配置文件中主要完成以下几件工作：

* 注册映射文件

* 注册实体类别名

* 注册数据源

  ![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6d83phskhj30ys0gcaj9.jpg)

  ![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6d843ssmaj30yo03eta7.jpg)

# Spring Boot的事物支持

## 定义工程

​	复制 07-mybatis 工程,并重命名为 08-transaction。 

​	当前工程完成在对用户注册时一次插入到 DB 中两条注册信息。若在插入过程中有发生异常,则已完成插入的记录回滚。 

## 修改启动类

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6e0nxxtbuj30zg0e8te7.jpg)

## 修改Service实现类

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6e0oa4fyjj31080newm4.jpg)

# Spring Boot对日志的控制

## logback日志技术介绍

​	Spring Boot 中使用的日志技术为 logback。其与 Log4J 都出自同一人,性能要优于 Log4J, 是 Log4J 的替代者。 

​	在 Spring Boot 中若要使用 logback,则需要具有 spring-boot-starter-logging 依赖,而该依 赖被 spring-boot-starter-web 所依赖,即不用直接导入 spring-boot-starter-logging 依赖。

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6e10be19ij30ze0dq7d4.jpg) 

## spring boot中使用logbook

### 添加配置属性

只需在核心配置文件中添加如下配置即可。

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6e1bcw4ovj30ze0ie0yl.jpg)

注意,在日志显示格式的属性值前面的 logs-是随意内容。在 yml 文件中的属性值若以%开头会报错,所以添加一些随意字符或将属性值用单引号引起来。在 properties 文件中不存在该问题。

### 添加配置文件

该文件名为 logback.xml,且必须要放在 src/main/resources 类路径下。

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6e1c4x6d8j30cq060t9r.jpg)

内容势力如下:

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6e1cwvsj2j30zs0jigrd.jpg)

# Spring Boot中使用Redis

## 定义工程

​	复制 08-transaction 工程,并重命名为 09-redisCache。 

​	当前工程完成让用户在页面中输入要查询学生的 id,其首先会查看 Redis 缓存中是否存 在,若存在,则直接从 Redis 中读取;若不存在,则先从 DB 中查询出来,然后再存放到 Redis 缓存中。但用户也可以通过页面注册学生,一旦有新的学生注册,则需要将缓存中的学生信息清空。根据 id 查询出的学生信息要求**必须是实时性**的,其适合使用**注解方式的 Redis 缓存**。 

​	同时,通过页面还可以查看到总学生数,但对其要求是差不多就行,**无需是实时性**的。 对于 Spring Boot 工程,其适合使用 **API 方式的 Redis 缓存**,<u>该方式方便设置缓存的到期时限</u>。 

## 修改pom文件

```xml
<!-- spring boot 与redis整合依赖 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>
```

## 修改主配置文件

在主配置文件中添加如下内容:

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6e1lioqzuj30z40dmq86.jpg)

## 修改实体类Student

​	由于要将查询的实体类对象缓存到 Redis,Redis 要求实体类必须序列化。所以需要实体类实现序列化接口。

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6e1mwkqc6j31040dgq7a.jpg)

## 修改代码

### 修改index页面

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6e1ptt0icj30xa0pi12e.jpg)

### 修改SomeHandler类

在其中添加两个处理器方法。

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6e1quy9tej30yg0g2aha.jpg)

### 修改Service接口

在 Service 接口中添加一个业务方法。

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6e1snm2voj30rm0f2n1g.jpg)

### 修改Service接口实现类

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6e1tdbyucj31060u04b6.jpg)

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6e1trfqwnj30z10u0n9x.jpg)

### 修改Dao接口

在其中添加两个方法。

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6e1uimmvcj30sm0h60xk.jpg)

### 修改映射文件

可以使用简单类名作为别名。

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6e1w60b9yj30zm0gsjym.jpg)

## 启动Redis

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6e1wwwv10j30ze08kgqv.jpg)

## 启动Linux主机

该示例需要启动三台 Redis 主机,三台 Redis 哨兵主机。

### 启动Redis集群

逐台启动三台 Redis 主机,即启动 Redis 集群。

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6e20w1spqj30zu0980zc.jpg)

### 登录Redis

逐台登录三台 Redis。

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6e21gwo8qj30z8062gnz.jpg)

### 查看Redis角色

逐台查看 Redis 的角色。

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6e22h56osj310a0bggu0.jpg)

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6e22o85cpj30ze0b0n2i.jpg)

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6e22vt3qnj30zk0cewk2.jpg)

### 启动sentinel集群

逐台启动三台 Sentinel 主机,即启动 Sentinel 集群。

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6e23h2awyj30zg0d6doe.jpg)

# Spring Boot下使用拦截器

​	在非 Spring Boot 工程中若要使用 SpringMVC 的拦截器,在定义好拦截器后,需要在 Spring 配置文件中对其进行注册。但 Spring Boot 工程中没有了 Spring 配置文件,那么如何使用拦 截器呢? 

​	Spring Boot 对于原来在配置文件配置的内容,现在全部体现在一个类中,该类需要继承 自 WebMvcConfigurationSupport 类,并使用@Configuration 进行注解,表示该类为一个 JavaConfig 类,其充当配置文件的角色。 

## 定义工程

复制 04-customConfig 工程,并重命名为 05-interceptor。

## 定义拦截器

```java
public class SomeInterceptor implements HandlerInterceptor {

	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception{
		System.out.println("执行Interceptor 拦截器："+request.getRequestURI());
		return true;
	}
}
```

## 定义处理器

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6e2bfqoetj30ya0l6gtb.jpg)

## 定义配置文件类

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6e2btpx2cj31020d2dmk.jpg)

# Spring Boot中使用Servlet

​	在 Spring Boot 中使用 Servlet,根据 Servlet 注册方式的不同,有两种使用方式。若使用的是 Servlet3.0+版本,则两种方式均可使用;若使用的是 Servlet2.5 版本,则只能使用配置类方式。

## 注解方式

### 创建工程

创建一个 Spring Boot 工程,并命名为 11-servlet01。

### 创建Servlet

​	这里在创建 Servlet 时需要注意,不能直接使用 Eclipse 中的向导直接创建 Servlet,无法创建。需要通过创建一个 class,让其继承自 HttpServlet 方式创建。然后在 Servlet 上添加`@WebServlet` 注解。

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6e2sjdmyvj30zu0ck0xn.jpg)

### 修改入口类

在入口类中添加 Servlet 扫描注解。

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6e2tk4t2gj30zm0cagqg.jpg)

## 配置类方式

### 创建工程

创建 spring boot 工程,并命名为 11-servlet02。

### 定义Servlet

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6e2uf807oj30zw0cgtdc.jpg)

### 定义配置类

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6e2upv9wij30zw0fg0zi.jpg)

# Spring Boot中使用Filter

​	在 Spring Boot 中使用 Filter 与前面的使用 Servlet 相似,根据 Filter 注册方式的不同,有两种使用方式。若使用的是 Servlet3.0+版本,则两种方式均可使用;若使用的是 Servlet2.5 版本,则只能使用配置类方式。

## 注解方式

### 使用工程

直接在 11-servlet01 工程上进行修改,不再创建新的工程。

### 创建Filter

这里在创建 Filter 时需要注意,不能直接使用 Eclipse 中的向导直接创建 Filter,无法创建。需要通过创建一个 class,让其实现 Filter 接口方式创建。然后在 Filter 上添加@WebFilter注解。

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6e2yr3jidj30zg0kydn4.jpg)

### 修改入口类

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6e2zjmc3cj30zw0dk0y6.jpg)

## 配置方式

### 使用工程

直接在 11-servlet02 工程上进行修改,不再创建新的工程。

### 定义Filter

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6e300qy0rj31000k6gsp.jpg)

### 修改配置类

在配置类中添加如下方法。

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6e30mq7bij30zq0aqaea.jpg)