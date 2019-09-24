# Thymeleaf简介

​	Thymeleaf[taɪm lif],百里香叶。**Thymeleaf 是一个流行的==模板引擎==,该模板引擎==采用 Java 语言开发==。模板引擎是为了==使用户界面与业务数据(内容)分离==而产生的,它==可以生成特定格式的文档==**。例如,用于网站的模板引擎就会生成一个标准的 HTML 文档。不同的语言体系 中都有自己的模板引擎,例如,Java 中常见的模板引擎有 `Velocity`、`Freemaker`、`Thymeleaf` 等。不同的模板引擎都会具有自己的特定的标签体系,而 **Thymeleaf 以 HTML 标签为载体, 在 HTML 的标签下实现对数据的展示**。 

​	Thymeleaf 本身与 SpringBoot 是没有关系的,但 SpringBoot 官方推荐使用 Thymeleaf 作为前端页面的数据展示技术,SpringBoot 很好地集成了这种模板技术。 

​	Thymeleaf 的官网为: http://www.thymeleaf.org 

​	 Spring Boot2.x 默认使用的是 Thymeleaf3.x 版本,而 Spring Boot1.x 则使用的是 Thymeleaf2.x 版本。 

# Spring Boot集成Thymeleaf

## 创建工程

创建一个 Spring Boot 工程,命名为 thymeleaf,并在创建工程时导入如下依赖。

![](http://ww4.sinaimg.cn/large/006y8mN6gy1g6e7vx234uj30zg0p4age.jpg)

## 定义配置文件

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6e7wj2vdcj30zq06s41k.jpg)

## 定义处理器

![](http://ww3.sinaimg.cn/large/006y8mN6gy1g6e7yqn7n9j30zy0ggdlo.jpg)

## 定义index.html页面

在 src/main/resources/templates 目录下定义 index.html 页面。

![](http://ww1.sinaimg.cn/large/006y8mN6gy1g6e7zdqouaj30mc0kyjvc.jpg)

在页面的\<html>标签中需要添加 Thymeleaf 的命名空间属性:

```xml
xmlns:th="http://www.thymeleaf.org"
```

![](http://ww2.sinaimg.cn/large/006y8mN6gy1g6e80ycy3dj30zk0lcjzm.jpg)

# Thymeleaf标准表达式

​	常用的 Thymeleaf 标准表达式有三种。标准表达式都是==用于获取代码中存放到 Model 中的属性值的==,只不过获取方式不同而已。 

​	以下举例均在前面的 thymeleaf 工程基础上直接修改,无需再创建新的 Module。 

## 变量表达式${...}

使用`${...}`括起来的表达式,称为变量表达式。该表达式的内容会显示在 HTML 标签体文本处。 

该表达式一般都是通过 th:text 标签属性进行展示的。 

### 修改处理器类

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-071735.png)

### 创建VO类

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-082658.png)

### 修改index页面

直接在页面中添加如下内容:

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-083935.png)

### 测试效果

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-084001.png)

## 选择表达式*{...}

​	选择表达式,也称为星号表达式,其是使用`*{...}`括起来的表达式。一般用于展示对象的属性。该表达式的内容会显示在HTML标签体文本处。但其需要与 `th:object` 标签属性联用, 先使用 `th:object` 标签选择了对象,再使用`*{...}`选择要展示的对象属性。该表达式可以有效降低页面中代码的冗余。 

​	不过,其也可以不与 `th:object` 标签联用,在`*{...}`中直接使用"对象.属性"方式,这种写法与变量表达式相同。 

​	该表达式一般都是通过 `th:text` 标签属性进行展示的。 

### 修改index页面

直接在页面中添加如下内容:

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-084410.png)

### 测试效果

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-084435.png)

## URL表达式@{...}

​	使用`@{...}`括起来,并且其中只能写一个绝对 URL 或相对 URL 地址的表达式,称为 URL 表达式。这个绝对/相对 URL 地址中一般是包含有动态参数的,需要结合变量表达式`${...}`进 行字符串拼接。 

​	`@{...}`中的 URL 地址具有三种写法。为了演示这三种写法的区别,先为当前工程添加一个上下文路径,然后直接在 index.html 文件中修改。 

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-084720.png)

### 以http协议开头的绝对地址

​	在进行字符串拼接时使用加号(+)连接,容易出错。但使用双竖线则无需字符串拼接,简单易读。但是,Idea 会对其中的问号(?)报错,不过其不影响运行

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-084930.png)

​	在页面通过查看源码可以看到其解析结果。当然,对于 and 符(&)Thymeleaf 会将其解析为实体形式(&amp;),但浏览器会对(&amp;)进行正确解析。

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-084945.png)

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-084957.png)

### 以/开头的相对地址

​	在 URL 表达式中,Thymeleaf 会将开头的斜杠(/)解析为当前工程的上下文路径 `ContextPath`,而浏览器会自动为其添加“http://主机名:端口号”,即其即为一个绝对路径。 

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-085117.png)

在页面通过查看源码可以看到其解析结果中已经添加了上下文路径。

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-085128.png)

而在页面则可以看到浏览器对其解析的结果已经添加了 http://localhost:8080。

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-085140.png)

### 不以/开头的相对地址

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-085351.png)

​	在页面通过查看源码可以看到其解析结果中是未添加任何东西的,即没有上下文路径。也就是说,其是相对于当前请求路径的一个相对地址。

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-085403.png)

​	而在页面则可以看到浏览器对其解析的结果已经添加了 http://localhost:8080/xxx/test,这是相对于当前请求路径的的一个相对地址的转换结果。

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-085435.png)

# Thymeleaf常见属性

## 官方在线文档

Thymeleaf 的属性很多,从官网的 Docs 模块的在线文档中可以看到。

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-090249.png)

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-090402.png)

5.2 标题为:设置指定属性的值。

## 逻辑运算相关属性

### th:if

该属性用于逻辑判断,类似于 JSTL 中的`<c:if/>`。

#### 修改处理器

在处理器中添加如下语句。

```java
model.addAttribute("gender", "male");
```

#### 修改index页面

在 index.html 文件中添加如下语句。

```jsp
<p th:if="${gender}=='male'">男</p><br>
```

#### 效果

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-090804.png)

### th:switch/th:case

该属性用于多分支判断,类似于 Java 中的 `Swith-Case` 语句。

#### 修改处理器

在处理器中添加如下语句。

```java
model.addAttribute("age", "35");
```

#### 修改index页面

​	在 index.html 文件中添加如下语句。 

​	一旦某个 case 与 switch 的值相匹配了,剩余的 case 则不再比较。`th:case="*"`表示默表示默认的 case,前面的 case 都不匹配时候执行该 case。 

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-091133.png)

### th:each

该属性用于遍历数组、List、Set、Map,类似于 JSTL 中的`<c:forEach/>`。

#### 遍历List

遍历数组、Set 与遍历 List 方式是相同的。

##### 修改处理器

在 Controller 中添加如下代码。

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-091514.png)

##### 修改index页面

前面的 stu 为当前遍历对象,而`${students}`为遍历的集合。

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-091550.png)

##### 效果

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-091601.png)

#### 遍历状态对象

​	如何获取到当前遍历的状态信息呢?例如,当前遍历的是第几个,当前遍历的是集合中索引号为多少的元素,这个索引号是奇数还是偶数等。这些状态信息被定义在一个状态对象中,而这个状态对象有两种获取方式,一种是在 `th:each` 中指定,一种是在当前遍历变量名后添加 Stat 后辍。

##### 常用的状态对象属性

* index: 当前遍历对象的索引号(从 0 开始计算)
* count: 当前遍历对象是第几个(从 1 开始计算)
* even/odd: 布尔值,当前遍历对象的索引号是否是偶数/奇数(从 0 开始计算)
* first/last: 布尔值,当前遍历对象是否是第一个/最后一个

##### 修改index

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-092059.png)

##### 效果

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-092121.png)

#### 遍历Map

需要清楚,Map 的键值对是一个 `Map.Entry` 对象。

##### 修改处理器

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-092358.png)

##### 修改index页面

Idea 对这个遍历对象的 key、value 属性会报错,但不影响运行。

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-092427.png)

##### 效果

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-092441.png)

## html标签相关

### th:text/th:utext

#### 修改处理器

```java
model.addAttribute("welcome","<h2>Thymeleaf,<br/>I'm learning.</h2>");
```

#### 修改index页面

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-092942.png)

#### 效果

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-092957.png)

### th:name/th:value

该属性用于获取标签动态 name 属性值,及标签的默认 value 值。

#### 修改处理器

```java
model.addAttribute("attrName", "age");
```

#### 修改index页面

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-093137.png)

#### 效果

在页面查看源码可以看到如下效果。

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-27-093226.png)

### URL路径相关

​	`th:action`、`th:src`、`th:href`,这三个都是与 URL 路径相关的属性。若这些 URL 中包含有动
态参数,则它们的值需要 URL 表达式`@{...}`与变量表达式`${...}`配合使用。下面以`<img/>`标签
中的 `th:src` 为例演示用法。

#### 创建目录放图片

​	在工程的 src/main/resources/static 目录下创建一个 Directory,命名为 images,并在其中存放入一张图片 car.jpg。 

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-013618.png)

#### 修改处理器

```java
model.addAttribute("photo", "car.jpg");
```

#### 修改index

```jsp
<img th:src="@{|/images/${photo}|}"/>
```

## css/js相关

### th:id/th:name

这两个属性可以获取标签的动态 id 与 name 属性,以便在 js 中使用。

### th:style

该属性用于获取标签的 css 动态样式。

### th:onclick

​	该属性用于获取标签的单击事件所触发的动态事件,即单击事件所触发的方法名。这些js 事件属性很多,都是以 th:on 开头。

## 内联属性th:inline

​	其应用场景是,在 HTML 某标签显示文本内部存在部分数据来自于动态数据,或 JS 内部需要使用动态数据的场景。在该场景中,可以使用`[[${...}]]`或`[(${...})]`的方式将动态数据嵌入到指定的文本或脚本内部。 

​	`th:inline` 的取值有四个:`text`, `javascript`、`css` 和 `non`。分别表示内联文本、内联脚本、 内联 css,与禁止内联,即不解析`[[${...}]]`。不过,`th:inline="text"`可以省略不写。 

### 需求场景描述

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-015121.png)

​	如上所示,我们的需求是,在”他的名字是“后面紧跟动态数据,但以上写法会使动态数据直接显示在`<p>`标签体中,而将”他的名字是“内容覆盖。

### 内联文本

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-015227.png)

运行效果是:

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-015245.png)

### 内联脚本

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-015256.png)

### 内联CSS

#### 修改index页面

先在 index 页面中添加一个`<div>`,要使用内联 CSS 将其背景色设置为红色。

```jsp
<div id="reddiv">
    我的背景色要变为红色
</div>
```

#### 修改处理器

```java
model.addAttribute("elementId", "reddiv");
model.addAttribute("bgColor", "red");
```

#### 再修改index页面

现在 index 页面中添加如下代码,使用内联 CSS 定义`<div>`的样式。此内联的写法 Idea不识别,但不影响运行。

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-020029.png)

## 万能属性th:attr

​	该标签只所以称为万能属性,其主要具有两个应用场景,这两个场景在官方文档中均有详细讲解。

### 为无定义的属性赋予动态参数

​	很多 HTML 标签的属性都有其对应的 Thymeleaf 的 th 命名空间中属性,但并不是所有的都存在对应属性。若没有其对应的 th 命名空属性,但又想让其获取动态值,就需要使用该属性了。 

5.1 标题为:为任意属性设置值。

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-020731.png) 

### 一次为多个属性赋予动态参数

若想一次为多个属性赋予动态参数,则可以使用该属性。

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-020821.png)

# Thymeleaf运算基础

## Thymeleaf字面量

> 字面量,即字面常量。Thymeleaf 中有四种字面量:文本、数字、布尔,及 null。

### 文本字面量

> 由单引号括起来的字符串,称为文本字面量。

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-021329.png)

### 数字字面量

> 数字字面量就是数字,可以是整数,也可以是小数。

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-021413.png)

### 布尔字面量

> 布尔字面量就是 true、false,也可以写为 TRUE、FALSE、True、False

#### 修改处理器

```java
model.addAttribute("isClose", false);
```

#### 修改index页面

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-021612.png)

### null字面量

> 表示空,其一般用于判断。若对象未定义,或对象已定义但其值为 null,则为 null 的判断结果为 true;若集合不为 null,但长度为 0,则为 null 的判断结果为 false。 

#### 修改处理器

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-021816.png)

#### 修改index页面

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-021858.png)

## Thymeleaf运算符

### 字符串拼接运算符

​	可以使用加号(+)进行字符串拼接,也可以将文本字面量与动态数据直接写入由双竖线
括起来的字符串内,此时无需使用加号进行连接。

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-022142.png)

### 算数运算符

> \+ , - , * , / , %,但不支持++与--运算。

### 关系运算符

以下两种写法均支持:

> \>, < , >= , <= , == , !=
>
> gt , lt , ge , le , eq , ne

### 逻辑运算符

> 非:not 或者 !
> 与:and
> 或:or

### 条件运算符

> ?:

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-022433.png)

# Thymeleaf内置对象

为了方便使用,Thymeleaf 中内置了很多对象,程序员可以直接使用。

## ServletAPI对象

通过`#reqest`、`#session`、`#servletContext` 可以获取到 `HttpServletRequest`、`HttpSession` 及 `ServletContext` 对象,然后就可以调用其相应的方法了。 

### 修改处理器

在处理器中再定义一个处理器方法。

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-030351.png)

### 定义页面

再定义一个页面 index2.html。

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-030444.png)

### 提交请求及运行效果

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-030523.png)

## 表达式实用对象

​	Tymeleaf  提供了大量的表达式实用对象,这些对象中存在大量的方法。这些方法直接使用`"#工具对象.方法"`的方式使用。其使用方式类似于 Java 中的静态方法。这些工具对象相关文档在官网文档的第 19 条中。

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-030722.png)

### 工具对象简介

​	这些工具对象都是以`#`开头,由于是对象,所以首字母是小写,且一般都是复数形式, 即一般都是以 s 结尾。

* \#executionInfo:获取当前Thymeleaf模板对象的执行信息。
* \#messages:获取外部信息。
* \#uris/#urls:URI/URL处理工具。
* \#conversions:类型转换工具。
* \#dates:日期处理工具。
* \#calendars:日历处理工具。
* \#numbers:数字处理工具。
* \#strings:字符串处理工具。
* \#Objects:对象处理工具。
* \#booleans:布尔值处理工具。
* \#arrays:数组处理工具。
* \#lists:List处理工具。
* \#sets:Set处理工具。
* \#maps:Map处理工具。
* \#aggregates:聚合处理工具,例如,对数组、集合进行求和、平均值等。
* \#ids:th:id属性处理工具。

### 应用举例

下面就简单举几个例子,演示一下用法。

#### 修改处理器

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-031256.png)

#### 定义页面

再定义一个页面 index3.html。

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-031346.png)

#### 效果

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-08-28-031359.png)