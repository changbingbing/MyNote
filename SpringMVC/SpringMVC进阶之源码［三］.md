# 源码篇

## 知识储备

### Servlet的生命周期方法

* **init：**`Servlet`对象创建之后调用

* **service：**`Servlet`对象被`HTTP`请求访问时调用

* **destroy：**`Servlet`对象销毁之前调用

### DispatcherServlet继承体系                                                    

 ![1545649169174](https://ws3.sinaimg.cn/large/006tNbRwly1fyjff0zl4zj307x03nq2v.jpg)

### InitializingBean接口介绍

`Spring`有两种`Bean`的初始化（**不是实例化**）方式：

*  一种是实现**`InitializingBean`**接口

*  一种是通过反射调用`bean`标签中的**`init-method`**属性指定的方法。


**不同点**：接口比配置效率高，但是配置消除了对`spring`的依赖。

* **`InitializingBean`**接口为`bean`提供了初始化方法的方式。
  * 它只包括**`afterPropertiesSet`**方法。
  * 凡是实现该接口的类，在**初始化`bean`**的时候会执行该方法。

* 实现`InitializingBean`接口与在配置文件中指定`init-method`有什么不同？
  * 系统**先调用**`afterPropertiesSet`方法
  * 然后**再调用**`init-method`中指定的方法。

这种`Bean`的初始化方式在`Spring`中是怎么实现的？

* **执行时机**：在创建`Bean`实例及设置完属性之后执行。

* 通过查看`spring`加载`bean`的源码类(**`AbstractAutowireCapableBeanFactory`**)可看出其中奥妙。

  `AbstractAutowireCapableBeanFactory`类中的`invokeInitMethods`讲解的非常清楚，源码如下：

![1545649196768](https://ws3.sinaimg.cn/large/006tNbRwly1fyjfflvpcgj30fe09m0vt.jpg)      

## 关键组件图解

### 处理器拦截器图解

![](https://ws4.sinaimg.cn/large/006tNbRwly1fyjg7nql2gj30ye0elmxp.jpg)

### 处理器映射器的注册和处理流程图解

![](https://ws2.sinaimg.cn/large/006tNbRwly1fyjg86wnhgj30ye0elt9f.jpg)

## DispatcherServlet主流程

### 初始化流程

* 初始化入口：`GenericServlet#init(config)`

![1545649207421](https://ws3.sinaimg.cn/large/006tNbRwly1fyjfg3nxdqj30fg02ydge.jpg)           

* 接下来准备调用`GenericServlet#init()`方法了，不过该方法它没有实现，而是被子类`HttpServletBean`给覆盖了，我们直接看看`HttpServletBean#init()`方法吧

 ![1545649212581](https://ws4.sinaimg.cn/large/006tNbRwly1fyjfgeclstj30fg0a776n.jpg)          

* 接下来调用`initServletBean()`，不过该方法需要去`HttpServletBean`的子类`FrameworkServlet`中去看看

![1545649220518](https://ws3.sinaimg.cn/large/006tNbRwly1fyjfgnbkbcj30fg080abn.jpg)           

* 方法调用到这里，我们终于知道`DispatcherServlet`初始化的主要工作是干嘛的了，就是为了创建`Spring`容器（其实容器内还要初始化一些组件）。接下来我们去看看`FrameworkServlet#initWebApplicationContext`方法

![1545649228502](https://ws2.sinaimg.cn/large/006tNbRwly1fyjfgzp8gjj30fe0a00w2.jpg)   

* `557`行以上是根据`springmvc.xml`配置文件创建`spring`容器，`onRefresh()`方法是初始化一些默认组件，比如`HandlerMapping`组件中的（`BeanNameURLHandlerMapping`），我们进入这个方法看看

![1545649237279](https://ws4.sinaimg.cn/large/006tNbRwly1fyjfhbrplbj30fg09gtay.jpg)           

* 看到了这个地方，我想大家已经明白入门程序为什么没有配置三大组件，`Spring`容器中却依然有这些组件了吧。这个地方初始化了很多默认配置，我们随便找个`initHandlerMapping`来了解一下它们是怎么实现的吧

![1545649245565](https://ws4.sinaimg.cn/large/006tNbRwly1fyjfhjglkaj30fg07n766.jpg)           

* 看看是如何加载默认策略的，进入`getDefaultStrategies()`方法看看

 ![1545649267530](https://ws1.sinaimg.cn/large/006tNbRwly1fyjfiiwdj4j30fg09k76x.jpg)          

* 那么`defaultStrategies`集合是如何初始化的呢？

![1545649256883](https://ws4.sinaimg.cn/large/006tNbRwly1fyjfib0fjoj30fg07nabw.jpg)           

* 至此，`DispatcherServlet`初始化工作就完成了。

### 访问处理流程

* 访问入口：`HttpServlet#service`方法（该方法的`request`和`response`参数不是`Http`开头的）

![1545649280546](https://ws2.sinaimg.cn/large/006tNbRwly1fyjfisqbzij30fg08275m.jpg)           

* 继续跟踪到另一个`service`方法（请求和响应都是`HTTP`开头），该方法根据请求的`method`分别调用相应的处理。

![1545649285913](https://ws3.sinaimg.cn/large/006tNbRwly1fyjfj1uga3j30fe0cvn0v.jpg)   

 ![1545649294559](https://ws4.sinaimg.cn/large/006tNbRwly1fyjfj9d8y4j30fe0d541k.jpg)  

* 那接下来我们应该去看看`doGet`和`doPost`方法是如何处理请求的？但是问题是我们应该看哪个类中的`doGet`和`doPost`方法呢？通过上面的继承体系分析得知，我们应该去`FrameworkServlet`类看看

![1545649307034](https://ws1.sinaimg.cn/large/006tNbRwly1fyjfjgj5z1j30fg07ejsl.jpg)           

* 到现在为止，我们还没有进入到我们的`DispatcherServlet`类，所以我们继续去`processRequest`方法看一下

![1545649314364](https://ws3.sinaimg.cn/large/006tNbRwly1fyjfjrbbfkj30fg092tb0.jpg)           

* 通过这个`doService`方法才正式进入到`DispatcherServlet`类中去执行

![1545649323084](https://ws1.sinaimg.cn/large/006tNbRwly1fyjfjzwzvbj30fe08ujuk.jpg)   

* 至此，我们终于找到了`DispatcherServlet`中最核心的一个方法：`doDispatch`，我们一起去看看

![1545649330867](https://ws4.sinaimg.cn/large/006tNbRwly1fyjfka0a3lj30fe09841a.jpg)   

![1545649339803](https://ws4.sinaimg.cn/large/006tNbRwly1fyjfkhlo6gj30fe09tacs.jpg)   

   

* 源码阅读到这里，最起码从主流程中我们可以得知`DispatcherServlet`是如何处理一个请求的了。但是我们看到现在，还没有看出来**`DispatcherServlet`是如何与三大组件进行联系**的。所以我们要分别找到三大组件与`DispatcherServlet`交互的地方。

* 处理器映射器与`DispatcherServlet`交互的代码：`getHandler`方法

![1545649348115](https://ws2.sinaimg.cn/large/006tNbRwly1fyjflwlsbuj30fg057q45.jpg)           

* 处理器适配器与`DispatcherServlet`交互的代码：`getHandlerAdapter`方法

![1545649352432](https://ws1.sinaimg.cn/large/006tNbRwly1fyjfm7gc7nj30fg0593zm.jpg)           

* 视图解析器与`DispatcherServlet`交互的代码需要多深入几层去了解，先进入`processDispatchResult`方法

![1545649359479](https://ws4.sinaimg.cn/large/006tNbRwly1fyjfmlsscnj30fg097di8.jpg)           

* 再进入`render`方法

![1545649367797](https://ws3.sinaimg.cn/large/006tNbRwly1fyjfmvxdwvj30fg07k40b.jpg)           

* 这个`resolveViewName`方法就是视图解析器和`DispatcherServlet`交互的方法

![1545649375186](https://ws1.sinaimg.cn/large/006tNbRwly1fyjfn5ppr2j30fg069wfr.jpg)           

* 到此，`DispatcherServlet`的主流程以及三大组件和它的联系，我们都已经搞清楚了，如果想继续深入了解三大组件的具体实现，可以自己尝试着去阅读以下。

### 拦截器处理流程

- 分析源码入口：`DispatcherServlet#doDispatcher`方法

![1545649619075](https://ws4.sinaimg.cn/large/006tNbRwly1fyjfo26j6lj30fg09nq5n.jpg)           

![image-20181226204320778](/Users/changbingbing/Library/Application Support/typora-user-images/image-20181226204320778.png)

- `preHandler`流程分析（`HandlerExecutionChain`类）

![1545649626849](https://ws4.sinaimg.cn/large/006tNbRwly1fyjfofyp0vj30fg052myv.jpg)           

- `postHandle`流程分析（`HandlerExecutionChain`类）

![1545649635996](https://ws1.sinaimg.cn/large/006tNbRwly1fyjfooashtj30fg03paax.jpg)           

- `afterCompletion`流程分析（`HandlerExecutionChain`类）

![1545649643778](https://ws1.sinaimg.cn/large/006tNbRwly1fyjfowg9kzj30fg053dh4.jpg)           

## 处理器映射器

![image-20181226204515165](/Users/changbingbing/Library/Application Support/typora-user-images/image-20181226204515165.png)

### 注册流程

本章节主要是分析注解方式的处理器映射器：

`RequestMappingHandlerMapping`：根据`@RequestMapping`注解查找处理器（`HandlerMethod`）



* **分析入口**：`RequestMappingHandlerMapping#afterPropertiesSet`方法

![1545649390622](https://ws4.sinaimg.cn/large/006tNbRwly1fyjfp9104jj30fe05cwg6.jpg)   

* 继续进入父类`AbstractHandlerMethodMapping#afterPropertiesSet`方法看看，其中又调用了`intitHandlerMethods`方法（**定义了映射器处理的主流程**）

![1545649397187](https://ws2.sinaimg.cn/large/006tNbRwly1fyjfpj3lq3j30fg09rjul.jpg)           

* 我们继续去看看`detectHandlerMethods`方法（核心处理方法）

![1545649404661](https://ws2.sinaimg.cn/large/006tNbRwly1fyjfpt1qlej30fe08racz.jpg)   

* 上面的方法处理逻辑主要分为两大步骤： 

```
1. 映射`Method`和`RequestMappingInfo`的关系

1. 映射关系：`RequestMappingInfo`与`URL`和`HandlerMethod`的关系。
```



* 我们先看**第一大步骤**，也就是调用`MethodIntrospector`.`selectMethods`方法

![1545649414884](https://ws1.sinaimg.cn/large/006tNbRwly1fyjfq1ve8gj30fe098tc5.jpg)       

* 我们去看一下`MetadataLookup`的匿名内部类实现中调用的`getMappingForMethod`方法是如何实现的？需要去`RequestMappingHandlerMapping`类中去查看该方法

![1545649481637](https://ws2.sinaimg.cn/large/006tNbRwly1fyjfqbw13aj30fe06b76s.jpg)

 ![1545649493052](https://ws3.sinaimg.cn/large/006tNbRwly1fyjfqjsozuj30fe068q55.jpg)  

* 至此第一大步骤我们阅读完了，接下来去看看**第二大步骤**，我们看看`AbstractHandlerMethodMapping#registerHandlerMethod`方法

![1545649514225](https://ws4.sinaimg.cn/large/006tNbRwly1fyjfqrsmjxj30fe01oglw.jpg)   

* 接下来，我们去看看`register`方法

![1545649524630](https://ws3.sinaimg.cn/large/006tNbRwly1fyjfqyi13xj30fe0aujuc.jpg)   

* 至此处理器映射器的初始化流程就完成了。

### 处理流程

* 分析入口：`DispatcherServlet#getHandler`方法

![1545649535123](https://ws3.sinaimg.cn/large/006tNbRwly1fyjfrc447cj30fg06nab4.jpg)           

* 接下来我们需要进入到具体的`AbstractHandlerMapping#getHandler`方法

![1545649544877](https://ws4.sinaimg.cn/large/006tNbRwly1fyjfrjhomzj30fg07pwh2.jpg)           

* 接下来我们重点看看`AbstractHandlerMethodMapping#getHandlerInternal`方法。

![1545649556306](https://ws1.sinaimg.cn/large/006tNbRwly1fyjfrovetfj30fg096mzv.jpg)           

* 接下来我们重点去看看`lookupHandlerMethod`方法（**核心方法**）

![1545649564184](https://ws4.sinaimg.cn/large/006tNbRwly1fyjfrwg3l4j30fg0830w3.jpg)           

![1545649570717](https://ws2.sinaimg.cn/large/006tNbRwly1fyjfs5un08j30fe02xq3e.jpg)   

* 我们继续去看看`addMatchingMappings`方法

![1545649577156](https://ws1.sinaimg.cn/large/006tNbRwly1fyjfsdixfkj30fg02s3z6.jpg)           

* 接着去`RequestMappingInfoHandlerMapping#getMatchingMapping`方法

![1545649584909](https://ws3.sinaimg.cn/large/006tNbRwly1fyjfskgmgcj30fg01odg2.jpg)           

* 最后去`RequestMappingInfo#getMatchingCondition`方法

![1545649593208](https://ws2.sinaimg.cn/large/006tNbRwly1fyjfsriklxj30fg08igo2.jpg)           

 

## 处理器适配器

![image-20181226205419794](/Users/changbingbing/Library/Application Support/typora-user-images/image-20181226205419794.png)

### 注册流程

* 分析源码入口：`RequestMappingHandlerAdapter#afterPropertiesSet`

![1545726242624](https://ws2.sinaimg.cn/large/006tNbRwly1fyjft57xu9j30fg05tgn5.jpg)           

* 以上方法，分别注册了**处理器增强类**的相关操作（`@ModelAttribute、@InitBinder、@ExceptionHandler`）、参数解析器、`initBinder`参数解析器、返回值处理器

* 这些注册的参数解析器和返回值处理器会在执行`Handler`方法时进行调用。

 

### 处理流程

![image-20181226205807206](/Users/changbingbing/Library/Application Support/typora-user-images/image-20181226205807206.png)

* 分析源码入口：`DispatcherServlet#doDispatcher`方法

  ![1545726382054](https://ws2.sinaimg.cn/large/006tNbRwly1fyjftfmvszj30fg04pq3m.jpg)

* 接着进入`AbstractHandlerMethodAdapter#Handle`方法

  ![1545726457319](https://ws3.sinaimg.cn/large/006tNbRwly1fyjftozvvcj30fg01x0t1.jpg)         

* 进入`RequestMappingHandlerAdapter#handleInternal`方法

  ![1545726466853](https://ws2.sinaimg.cn/large/006tNbRwly1fyjfu37ln5j30fg0biq5u.jpg)         

* 进入`invokeHandlerMethod`方法

​    ![1545726519592](https://ws4.sinaimg.cn/large/006tNbRwly1fyjfuf6c7pj30fg08uwhs.jpg)

![1545726528131](https://ws2.sinaimg.cn/large/006tNbRwly1fyjfun33zdj30fg09ago9.jpg)       

​           

* 进入`ServletInvocableHandlerMethod#invokeAndHandle`方法

  ![1545726549877](https://ws1.sinaimg.cn/large/006tNbRwly1fyjfuv2tr0j30fg095dht.jpg)         

* 到此为止，我们已经看完处理器适配器的处理流程了。

### 参数绑定流程

* 分析入口：`InvocableHandlerMethod#invokeForRequest`方法

   ![1545726632465](https://ws1.sinaimg.cn/large/006tNbRwly1fyjfv3jnadj30fg05g75w.jpg)        

* 进入`getMethodArgumentValues`方法

   ![1545726639471](https://ws1.sinaimg.cn/large/006tNbRwly1fyjfvcxotpj30fg09t76s.jpg)        

* 重点看看`HandlerMethodArgumentResolverComposite#resolveArgument`方法

  ![1545726649820](https://ws4.sinaimg.cn/large/006tNbRwly1fyjfvozvfrj30fg0910vf.jpg)         

* 此处需要根据具体参数类型调用相应的解析器，其中**简单类型**调用的都是**`AbstractNamedValueMethodArgumentResolver`**类中的方法；**`POJO`类型**调用的都是**`ModelAttributeMethodProcessor`**类中的方法。我们就以简单类型（**对应`AbstractNamedValueMethodArgumentResolver`**）为例给大家讲解一下具体的参数绑定过程

![1545726664030](https://ws2.sinaimg.cn/large/006tNbRwly1fyjfvy64vaj30fe098n0r.jpg)      

* 我们进入`RequestParamMethodArgumentResolver#resolveName`方法看看

![1545726748905](https://ws4.sinaimg.cn/large/006tNbRwly1fyjfw5qtauj30fg08fwgg.jpg)           

* 至于参数绑定器的源码比较复杂，我们就不深入追踪了，我们只需要知道**参数转换**方式有两种处理方式：**`PropertyEditor`和`ConversionService`**，以及它们处理参数转换的区别就行了。下面的代码是在**`TypeConverterDelegate`**类中的（至于怎么过来的，同学们自己跟踪一下试试吧）

![1545726760897](https://ws1.sinaimg.cn/large/006tNbRwly1fyjfwdie4rj30fe06ltas.jpg)   



### 返回值处理流程

* 分析入口：`ServletInvocableHandlerMethod#invokeAndHandle`方法

  ![1545726863555](https://ws2.sinaimg.cn/large/006tNbRwly1fyjfwnz0myj30fe09c76i.jpg) 

* 返回值由`this.returnValueHandlers`进行处理，这个成员变量的类型是**`HandlerMethodReturnValueHandlerComposite`**，它继承自`HandlerMethodReturnValueHandler`类，该类有两个方法：

 ![1545726872388](https://ws4.sinaimg.cn/large/006tNbRwly1fyjfww7of4j30fe03x75h.jpg)  

* 我们进入`HandlerMethodReturnValueHandlerComposite#handleReturnValue`方法

 ![1545726879530](https://ws4.sinaimg.cn/large/006tNbRwly1fyjfx3y1ovj30fg07emz8.jpg)          

* 至此返回值处理流程我们就分析完了

### ResponseBody注解解析

* 如果返回值使用`@ResponseBody`注解，那么由它注解的返回值会使用**`RequestResponseBodyMethodProcessor`**类进行返回值处理。

 ![1545726940897](https://ws3.sinaimg.cn/large/006tNbRwly1fyjfxbe1g9j30fg04mwfu.jpg)          

* 进入`AbstractMessageConverterMethodProcessor#writeWithMessageConverters`方法，这个方法的处理逻辑比较长，我们挑重点的代码进行阅读
   ![1545726976731](https://ws2.sinaimg.cn/large/006tNbRwly1fyjfxlqdv3j30fg082wh8.jpg)

![1545726986981](https://ws3.sinaimg.cn/large/006tNbRwly1fyjfxtbe05j30fg08lacw.jpg)        

​           

* 继续深入`AbstractGenericHttpMessageConverter#write`方法

   ![1545727008354](https://ws3.sinaimg.cn/large/006tNbRwly1fyjfxzde1kj30fe0a1q5a.jpg)

* 进入`writeInternal`方法

   ![1545727023207](https://ws1.sinaimg.cn/large/006tNbRwly1fyjfy5udyyj30fe08uq5q.jpg)

![1545727031763](https://ws1.sinaimg.cn/large/006tNbRwly1fyjfyd34gwj30fe07940d.jpg)

## 视图解析器

### 注册流程

 

### 处理流程

* 分析入口：`DispatcherServlet`类的`1008`行代码

![1545727319203](https://ws2.sinaimg.cn/large/006tNbRwly1fyjfyrb99jj30fe00kq2w.jpg)   

* 进入`processDispatchResult`方法

![1545727326728](https://ws4.sinaimg.cn/large/006tNbRwly1fyjfz21ol4j30fe09e416.jpg)   

* 我们此时注意目的是为了解决视图解析，所以进入`render`方法看看

![1545727335370](https://ws3.sinaimg.cn/large/006tNbRwly1fyjfzbilysj30fg0820uw.jpg)           

* 进入`resolveViewName`方法

![1545727343461](https://ws1.sinaimg.cn/large/006tNbRwly1fyjfzjlhpnj30fg05dgmr.jpg)           

* 接下来进入`ViewResolverComposite#resolveViewName`方法

![1545727353308](https://ws3.sinaimg.cn/large/006tNbRwly1fyjfztd5znj30fg03uwf3.jpg)           

* 接下来进入`AbstractCachingViewResolver`方法

![1545727365026](https://ws4.sinaimg.cn/large/006tNbRwly1fyjg00yzwtj30fg0dk411.jpg)           

* 进入`UrlBasedViewResolver#createView`方法

![1545727372335](https://ws3.sinaimg.cn/large/006tNbRwly1fyjg07zu35j30fg07dabw.jpg)           

* 经过几波周转，最终请求进入到`UrlBasedViewResolver#buildView`方法，获得基础视图对象

![1545727380828](https://ws1.sinaimg.cn/large/006tNbRwly1fyjg0gdy4ij30fe0b077i.jpg)   

* 最终由该类的以下方法获取真正的视图对象

![1545727389360](https://ws1.sinaimg.cn/large/006tNbRwly1fyjg0mnsr0j30fe03l0tg.jpg)   

