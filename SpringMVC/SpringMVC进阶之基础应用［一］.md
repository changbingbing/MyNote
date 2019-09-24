# SpringMVC简介

​	大部分java应用都是web应用，展现层是web应用最为重要的部分。Spring为展现层提供了一个优秀的web框架——**Spring MVC**。和众多其他web框架一样，它基于**MVC的设计理念**，此外，它采用了**松散耦合可插拔**组件结构，比其他MVC框架更具扩展性和灵活性。

　　SpringMVC通过一套MVC注解，让POJO成为处理请求的控制器，无需实现任何接口，同时，SpringMVC还支持`REST`风格的URL请求。此外，SpringMVC在**数据绑定**、**视图解析**、**本地化处理**及**静态资源处理**上都有许多不俗的表现。它在框架设计、扩展性、灵活性等方面全面超越了`Struts`、`WebWork`等MVC框架，从原来的追赶者一跃成为MVC的领跑者。

　　SpringMVC框架围绕`DispatcherServlet`这个核心展开，`DispatcherServlet`是SpringMVC框架的总导演、总策划，它**负责截获请求并将其分派给相应的处理器处理**。

## Spring体系简介

![](https://ws2.sinaimg.cn/large/006tNc79ly1fytsjxii0mj30it0dtwg5.jpg)

## 回顾MVC设计模式

![](https://ws1.sinaimg.cn/large/006tNc79ly1fytskpi24vj30hc07iglz.jpg)

1. 用户将请求发送到控制器，控制器本身没有处理业务逻辑的能力；
2. 将请求委托给模型做处理，模型将处理结果返回到控制器；
3. 控制器将模型对视图做渲染；
4. 将最终的视图（`html`）返回给用户；

　　注意：`MVC`不是java独有的，其他语言也有，它是一种模式！它也不是`B/S`独有的，`C/S`也有！

## SpringMVC整体架构

![](https://ws1.sinaimg.cn/large/006tNc79ly1fytsmd6h58j30kp0bj75l.jpg)

1. 用户发起请求到**前端控制器**（`DispatcherServlet`），前端控制器没有能力处理业务逻辑；
2. 通过`HandlerMapping`查找**模型**（`Controller`、`Handler`）；
3. 返回**执行链**，执行链包含了2部分内容，**`Handler`对象**以及**拦截器（组）**：n个`HandlerInterceptor`；
4. 根据处理器`Handler`找到**适配器**`HandlerAdapter`，再通过`HandlerAdapter`去适配、执行模型（`Handler`）
5. 适配器调用`Handler`对象处理业务逻辑；
6. 模型处理完业务逻辑，返回**`ModelAndView`对象**，`view`不是真正的视图对象，而是**视图名称**；
7. 将`ModelAndView`对象返回给前端控制器；
8. 前端控制器通过视图名称经过视图解析器（`ViewResolver`）查找视图对象；
9. 返回视图对象；
10. 前端控制器渲染视图；
11. 返回给前端控制器；
12. 前端控制器将视图（`html`、`json`、`xml`、`Excel`）返回给用户；

## 第一个SpringMVC程序

1. 配置SpringMVC框架入口`web.xml`

   ```xml
   <!-- SpringMVC框架的入口 -->
   <servlet>
   <servlet-name>springmvc</servlet-name>
   <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
       <load-on-startup>1</load-on-startup>
       <!-- 默认查找配置文件规则  /WEB-INF/servletName-servlet.xml -->
   </servlet>
   <servlet-mapping>
       <servlet-name>springmvc</servlet-name>
       <!-- 所有请求以*.do会进入MVC框架(.do是struts1的后缀) -->
       <url-pattern>*.do</url-pattern>
   </servlet-mapping>
   ```

2. 配置核心组件`springmvc-servlet.xml`

   1. 配置`HandlerMapping`

      ```xml
      <!-- 注册HandlerMapping -->
      <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
      ```

      通过`beanName`查找，`/hello.do` =>  `cn.itcast.springmvc.controller.HelloController`

   2. 配置适配器

      ```xml
      <!-- 注册简单适配器 -->
      <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
      ```

   3. 定义Controller

      ```xml
      <!-- 自定义Handler -->
      <bean name="/hello.do" class="cn.itcast.springmvc.controller.HelloController"/>
      ```

      > 要指定名称：*/hello.do*

   4. 定义视图解析器

      ```xml
      <!-- prefix="/WEB-INF/jsp/", suffix=".jsp", viewname="test" -> "/WEB-INF/jsp/test.jsp"  -->
      <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
          <!-- 前缀 -->
          <property name="prefix" value="/WEB-INF/views/"/>
          <!-- 后缀 -->
          <property name="suffix" value=".jsp"/>
      </bean>
      ```

      > 设置前缀和后缀，得到具体视图路径；
      >
      > `prefix="/WEB-INF/jsp/", suffix=".jsp", viewname="test"` -> `"/WEB-INF/jsp/test.jsp"`

3. 测试
   <http://127.0.0.1/hello.do>

   ![](https://ws2.sinaimg.cn/large/006tNc79ly1fytt21tvvpj30cq02gmx9.jpg)

4. 执行过程分析
   ![](https://ws2.sinaimg.cn/large/006tNc79ly1fytt2u9niaj30ng0aymyl.jpg)

5. SpringMVC的默认装载组件 

   > 在org.springframework.web.servlet/DispatcherServlet.properties 默认配置

   ```properties
   # Default implementation classes for DispatcherServlet's strategy interfaces.
   # Used as fallback when no matching beans are found in the DispatcherServlet context.
   # Not meant to be customized by application developers.
   
   org.springframework.web.servlet.LocaleResolver=org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver
   
   org.springframework.web.servlet.ThemeResolver=org.springframework.web.servlet.theme.FixedThemeResolver
   
   org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
       org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping
   
   org.springframework.web.servlet.HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
       org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
       org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter
   
   org.springframework.web.servlet.HandlerExceptionResolver=org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerExceptionResolver,\
       org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,\
       org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver
   
   org.springframework.web.servlet.RequestToViewNameTranslator=org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator
   
   org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver
   
   org.springframework.web.servlet.FlashMapManager=org.springframework.web.servlet.support.SessionFlashMapManager
   ```



## 第一个SpringMVC注解程序

1. 创建Controller

   ```java
   @RequestMapping("/demo") //标识请求进入Controller的url path
   @Controller //用来标识这个一个Controller（Handler）
   public class Demo1Controller {
   
       @RequestMapping("/show1") //标识请求进入Controller的url path
       public ModelAndView show1() {
           ModelAndView mv = new ModelAndView();
           mv.setViewName("demo1");
           mv.addObject("msg", "我的第一个注解程序!");
           return mv;
       }
   }
   ```

2. 配置扫描器

   ```xml
   <!-- 扫描包，使@Controller生效 -->
   <context:component-scan base-package="cn.itcast.springmvc.controller"/>
   ```

3. 配置视图解析器

   ```xml
   <!-- prefix="/WEB-INF/jsp/", suffix=".jsp", viewname="test" -> "/WEB-INF/jsp/test.jsp"  -->
   <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
       <!-- 前缀 -->
       <property name="prefix" value="/WEB-INF/views/"/>
       <!-- 后缀 -->
       <property name="suffix" value=".jsp"/>
   </bean>
   ```

4. 创建jsp页面
   创建页面：/WEB-INF/views/demo1.jsp

   ```jsp
   <%@ page language="java" contentType="text/html; charset=UTF-8"
       pageEncoding="UTF-8"%>
   <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
   <html>
   <head>
   <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
   <title>Insert title here</title>
   </head>
   <body>
   <h1>${msg}</h1>
   </body>
   </html>
   ```

5. 测试
   <http://127.0.0.1/demo/show1.do>
   ![](https://ws2.sinaimg.cn/large/006tNc79ly1fytt8eplxpj30au026dfw.jpg)

## HandlerMapping和HandlerAdapter

```xml
<!-- 推荐使用注解的HandlerMapping -->
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>
    
<!-- 推荐使用注解的适配器 -->
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>
```

**其实这两个适配器也是不需要配置的，可以只配置下面提到的mvc注解驱动即可！**

## 使用MVC注解驱动

* 在servlet配置文件中加入注解驱动标签即可：

  ```xml
  <!-- mvc的注解驱动 -->
  <mvc:annotation-driven/>
  ```

* 注解驱动原理
  `org.springframework.web.servlet.config.AnnotationDrivenBeanDefinitionParser`
  ![](https://ws3.sinaimg.cn/large/006tNc79ly1fyttn67cjdj30sf0ek0xe.jpg)



# @Requestmapping映射

​	在`SpringMVC`中的众多`Controller`以及每个`Controller`的众多方法，请求是如何映射到具体的处理方法上？这个就是靠`@RequestMapping`完成的。

* `@RequestMapping`既可以定义在类上也可以定义在方法上
* 请求映射的规则是：**类上的`@RequestMapping.value` +** **方法上的`@RequestMapping.value`**
  `@RequestMapping(value = "/hello")` 和 `@RequestMapping("/hello")` 等效;

**映射**

1. 标准URL映射
2. Ant风格的URL映射
3. 占位符映射
4. 限制请求方法映射
5. 限制参数映射

## 标准URL映射

标准URL映射是最简单的一种映射

> 例如：**@RequestMapping("/hello")  或 @RequestMapping(value="/hello")**
>
> 请求url：<http://localhost/hello.action>

## Ant风格的URL映射

![](https://ws1.sinaimg.cn/large/006tNc79ly1fyttsh3zqbj30d2083gm9.jpg)

```java
/**
* Ant风格的通配符映射！
* 
* @return
*/
@RequestMapping(value = "/demo/**/show3")
public ModelAndView show3() {
    ModelAndView mv = new ModelAndView();
    mv.setViewName("hello");
    mv.addObject("msg", "Ant风格的通配符(**)映射！");
    return mv;
}
```

## 占位符映射

Url中可以通过一个或多个`{xxxx}`占位符映射。通过*`@PathVariable("xxx")`*绑定到方法的入参中。

> 例如：**`@RequestMapping("/user/{userId}/query")`**
>
> 请求URL：<http://localhost/user/8/query>

 

RESTFul风格：<http://127.0.0.1/demo/123/show4>.....

```java
/**
* url占位符
* 这里参数id改为String，也可以传中文，但会有乱码，所以一般传递中文参数会选择post提交
* @return
*/
@RequestMapping(value = "/demo/{id}/show4")
public ModelAndView show4(@PathVariable("id") Long id) {
    ModelAndView mv = new ModelAndView();
    mv.setViewName("hello");
    mv.addObject("msg", "url占位符测试 id = " + id + "！");
    return mv;
}
```

## @PathVariable小误区

![](https://ws2.sinaimg.cn/large/006tNc79ly1fyttv5yjxmj30gz0agmyx.jpg)

![](https://ws2.sinaimg.cn/large/006tNc79ly1fyttvc9vvfj30k60kigpi.jpg)

> 小结：只有在开发时debug的环境下`@PathVariable(xxxx)` 可以简写为：`@PathVariable`！因此**不要简写**！

## 限定请求方式的映射

在Http请求中最常用的请求方法是`GET`、`POST`，还有其他的一些方法，`DELET`、`PUT`、`HEAD`等。

> 例如：
>
> `@RequestMapping(value = "/{userId}/query",method=RequestMethod.GET)`
>
> `@RequestMapping(value = "/{userId}/query",method={RequestMethod.GET,RequestMethod.POST})`

```java
/**
     * 限制请求方法GET、POST
     * 
     * @return
     */
    @RequestMapping(value = "/demo/test/show5", method = RequestMethod.POST)
    public ModelAndView show5() {
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg", "你使用的是POST请求！");
        return mv;
    }

    /**
     * 限制请求方法GET、PUT
     * 
     * @return
     */
    @RequestMapping(value = "/demo/test/show6", method = { RequestMethod.GET, RequestMethod.PUT })
    public ModelAndView show6() {
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg", "你使用的是GET或PUT请求！");
        return mv;
    }
```

## 限定参数映射

设定请求的参数来映射URL

> 例如：*`@RequestMapping(value="/query",params="userId")`，*要求请求中必须带有userId参数。

参数的限制规则如下：

* `params="userId"` 请求参数中必须包含userId

* `params="!userId"` 请求参数中不能包含userId

* `params="userId!=1"` 请求参数必须包含userId，但其值不能为1

* `params={"userId","name"}` 必须包含userId和name参数

```java
/**
     * 限制请求参数
     * 
     * @return
     */
    @RequestMapping(value = "/demo/test/show7", params = "id")
    public ModelAndView show7(@RequestParam("id") String id) {
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg", "现在请求参数  id = " + id + "！");
        return mv;
    }

    /**
     * 限制多个请求参数
     * 
     * @return
     */
    @RequestMapping(value = "/demo/test/show8", params = { "id", "name" })
    public ModelAndView show8(@RequestParam("id") String id, @RequestParam("name") String name) {
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg", "现在请求参数  id = " + id + "  name = " + name + "！");
        return mv;
    }
```

# 获取参数

## 绑定Servlet内置对象

在Controller中获取Servlet的内置对象（Request、Response、Session）是非常简单的，如下：

```java
public void test(HttpServletRequest request){……}

public void test(HttpServletResponse response){……}

public void test(HttpSession session){……}

public void test(Model model){……}//Model:数据模型，主要存储要返回到客户端的数据的，功能类似于Request对象。
```

```java
/**
     * 获取Servlet内置对象
     * @param request
     * @param response
     * @param session
     * @return
     */
    @RequestMapping(value = "/demo/test/show9")
    public ModelAndView show9(HttpServletRequest request,HttpServletResponse response,HttpSession session){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("servlet");
        
        StringBuilder sb = new StringBuilder();
        sb.append("request = " + request);
        sb.append("<br/>response = " + response);
        sb.append("<br/>session = " + session);
        
        request.setAttribute("param1", "在Request中参数!");
        
        mv.addObject("msg", sb);
        return mv;
}
```

结果：

![](https://ws4.sinaimg.cn/large/006tNc79gy1fyulf1g333j30m7058js3.jpg)

## @PathVariable

通过*@PathVariable*可以绑定占位符参数到方法参数中，可以认为是**url中？前面的参数**，例如：**`@PathVariable("userId") Long userId`**

*参考***@RequestMapping映射-占位符映射**部分！

## @RequestParam

`@RequestParam`有三个参数(**url中？后面的参数**)：

1.  `value`*：参数名；*
2. `required`：是否必需，默认为`true`，表示请求参数中必须包含该参数，如果不包含抛出异常。
3. `defaultValue`：默认参数值，如果设置了该值自动将`required`设置为`false`，如果参数中没有包含该参数则使用默认值。

示例：**`@RequestParam(value = "userId", required = false, defaultValue = "1")`**

```java
/**
     * 获取？后面传递的参数
     * 
     * @return
     */
    @RequestMapping(value = "/demo/test/show10")
    public ModelAndView show10(@RequestParam(value = "id", required = false, defaultValue = "000")String id){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg", "现在请求参数  id = "+id+" ！");
        return mv;
    }
```



## @CookieValue

在SpringMVC中获取`cookie`值更加方便了，`@CookieValue`可以绑定`cookie`值，和`@RequestParam`一样有三个参数，使用方法：

`public void test(@CookieValue("JSESSIONID") String jsessionid)`

```java
/**
     * 获取cookie中数据
     * 
     * @return
     */
    @RequestMapping(value = "/demo/test/show11")
    public ModelAndView show11(@CookieValue("JSESSIONID")String JSESSIONID){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg", "JSESSIONID  = "+JSESSIONID+" ！");
        return mv;
}
```

结果：

![](https://ws2.sinaimg.cn/large/006tNc79gy1fyulndzox7j30ek01qmx6.jpg)

## POJO对象绑定参数

SpringMVC会将请求过来的参数名和POJO实体中的属性名进行匹配，如果名称一致，将把值填充到对象中。

```java
/**
     * POJO对象绑定
     * 
     * @return
     */
    @RequestMapping(value = "/demo/test/show12")
    public ModelAndView show12(User user){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg", "user  = "+user+" ！");
        return mv;
}
```

<http://127.0.0.1/demo/test/show12?id=1&userName=123&password=111&name=abc>

![](https://ws4.sinaimg.cn/large/006tNc79gy1fyulogw006j30np02b0sz.jpg)

## JAVA的基本数据类型绑定

Spring对Java的基本数据类型的转换支持的非常多，基本能满足日常开发需要，默认支持的数据类型在`org.springframework.beans.PropertyEditorRegistrySupport`中定义。

1. testForm.html

   ```html
   <!DOCTYPE>
   <form action="/demo/test/show13.action" method="post">
       <div>姓名:</div>
       <div><input name="name" value="张三"/></div>
       <div class="clear"></div>
       <div>年龄:</div>
       <div><input name="age" value="20"/></div>
       <div class="clear"></div>
       <div>收入:</div>
       <div><input name="income" value="100000"/></div>
       <div class="clear"></div>
       <div>结婚:</div>
       <div>
       <input type="radio" name="isMarried" value="true" checked="checked"/>是
       <input type="radio" name="isMarried" value="false"/>否</div>
       <div class="clear"></div>
       <div>兴趣:</div>
       <div>
       <input type="checkbox" name="interests" value="听歌" checked="checked"/>听歌
       <input type="checkbox" name="interests" value="书法" checked="checked"/>书法
       <input type="checkbox" name="interests" value="看电影" checked="checked"/>看电影
       </div>
       <div class="clear"></div>
       <div><input type="submit" value="提交表单"/></div>
   </form>
   ```

2. java测试代码

   ```java
   @RequestMapping("/demo/test/show13")
       @ResponseStatus(value = HttpStatus.OK)// 无需跳转页面，直接返回200状态
       public void show13(String name, Integer age, Double income, Boolean isMarried, String[] interests) {
           System.out.println("简单数据类型绑定=========");
           System.out.println("名字:" + name);
           System.out.println("年龄:" + age);
           System.out.println("收入:" + income);
           System.out.println("已结婚:" + isMarried);
           System.out.println("兴趣:");
           for (String interest : interests) {
               System.out.println(interest);
           }
           System.out.println("====================");
       }
   ```

## 集合List绑定

List的绑定需要将List对象包装到一个类中才能绑定。

![](https://ws3.sinaimg.cn/large/006tNc79gy1fyuls9lo8tj30ga041jry.jpg)

![](https://ws2.sinaimg.cn/large/006tNc79gy1fyulsrm4ihj308205e3yi.jpg)

# SpringMVC和Struts2的区别(面试)

1. SpringMVC的入口是**Servlet**，Struts2的入口是**Filter**，两者的实现机制不同；
2. SpringMVC**基于方法**设计，传递参数是通过方法形参，其实现是单例模式（也可以改为多例，推荐用单例），Struts2**基于类**设计，传递参数是通过类的属性，只能是多例实现，性能上SpringMVC更高一些。
3. 参数传递方面，Struts2是用类的属性接收的，也就是在多个方法间共享，而SpringMVC基于方法，多个方法间不能共享。

# 视图与视图解析器

## JSP和JSTL视图解析器

`InternalResourceViewResolver`默认使用的是`JSTL`解析器，要想使用`JSTL`标签需要导入`JSTL`的依赖。

1. 导入`JSTL`的依赖

   ```xml
   <dependency>
          <groupId>jstl</groupId>
          <artifactId>jstl</artifactId>
          <version>1.2</version>
   </dependency>
   ```

2. user.jsp

   ```jsp
   <%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
   <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
   <!DOCTYPE html>
   <html>
       <head>
           <title>JSTL Demo</title>
       </head>
       
       <body>
           <table>
               <thead>
               <tr>
                   <th>ID</th>
                   <th>UserName</th>
                   <th>Name</th>
                   <th>Age</th>
               </tr>
               </thead>
               <tbody>
                   <c:forEach items="${users}" var="user">
                       <tr>
                           <td>${user.id}</<td>
                           <td>${user.userName}</<td>
                           <td>${user.name}</<td>
                           <td>${user.age}</<td>
                       </tr>
                   </c:forEach>
               </tbody>
           </table>
       </body>
   </html>
   ```

3. 构造`List<User>`

   ```java
   /**
        * 测试JSTL标签
        * @return
        */
       @RequestMapping("/demo/test/show14")
       public ModelAndView show14() {
           ModelAndView modelAndView = new ModelAndView("users");//通过构造直接赋视图名
           List<User> users = new ArrayList<User>();
   
           for (int i = 0; i < 10; i++) {
               User user = new User();
               user.setAge(20 + i);
               user.setCreated(new Date());
               user.setId(Long.valueOf(i));
               user.setName("name_" + i);
               user.setPassword("123456");
               user.setSex(i / 2);
               user.setUpdated(user.getCreated());
               user.setUserName("userName_" + i);
               users.add(user);
           }
           modelAndView.addObject("users", users);
           return modelAndView;
       }
   ```

## @ResponseBody

![](https://ws4.sinaimg.cn/large/006tNc79gy1fyum3dunukj30fd06mq3m.jpg)

**补充**：`@ResponseBody`注解，除了可以返回`json`，还可以返回`xml`等其他视图，但是用的最多的是返回`json`

```java
/**
     * 测试@ResponseBody
     * 
     * @return
     */
    @RequestMapping(value = "/demo/test/show15")
    @ResponseBody
    public User show15() {
        User user = new User();
        Integer i = 0;
        user.setAge(20 + i);
        user.setCreated(new Date());
        user.setId(Long.valueOf(i));
        user.setName("name_" + i);
        user.setPassword("123456");
        user.setSex(i / 2);
        user.setUpdated(user.getCreated());
        user.setUserName("userName_" + i);
        return user;
    }

    /**
     * 测试@ResponseBody，返回集合
     * 
     * @return
     */
    @RequestMapping(value = "/demo/test/show16")
    @ResponseBody
    public List<User> show16() {
        List<User> users = new ArrayList<User>();

        for (int i = 0; i < 10; i++) {
            User user = new User();
            user.setAge(20 + i);
            user.setCreated(new Date());
            user.setId(Long.valueOf(i));
            user.setName("name_" + i);
            user.setPassword("123456");
            user.setSex(i / 2);
            user.setUpdated(user.getCreated());
            user.setUserName("userName_" + i);
            users.add(user);
        }
        return users;
    }
```



## @RequestBody

![](https://ws1.sinaimg.cn/large/006tNc79gy1fyum62o2c5j30fc06bq3g.jpg)

1. 示例代码：

   ```java
   /**
        * 测试@RequestMapping，将json字符转化为java对象
        * @param user
        * @return
        */
       @RequestMapping(value = "/demo/test/show17")
       public ModelAndView show17(@RequestBody User user) {
           ModelAndView mv = new ModelAndView();
           mv.setViewName("hello");
           mv.addObject("msg", "user  = " + user + " ！");
           return mv;
   }
   ```

2. 测试
   ![](https://ws2.sinaimg.cn/large/006tNc79gy1fyum7cm3rvj30kj0dvwfp.jpg)

   ![](https://ws2.sinaimg.cn/large/006tNc79gy1fyum8krsp9j30lm02j0t9.jpg)

# 文件上传

上传原理：首先先保存到一个临时文件中，然后将这个临时文件写到目标文件中，最后将那个临时文件删除！



# 拦截器

## 理解HandlerExecutionChain

## 拦截的执行过程

## 自定义拦截器

## 拦截器总结