# 前置概念

## 服务熔断简介

### 雪崩效应

![image-20191008163014037](../assets-images/image-20191008163014037.png)

![image-20191008163048038](../assets-images/image-20191008163048038.png)

### 服务雪崩

![image-20191008171427035](../assets-images/image-20191008171427035.png)

上图是用户请求的多个服务(A,H,I,P)均能正常访问并返回的情况。

![image-20191008171456981](../assets-images/image-20191008171456981.png)

上图为请求服务 I 出现问题时,一个用户请求被阻塞的情况。

![image-20191008171538134](../assets-images/image-20191008171538134.png)

上图为大量用户请求服务 I 出现异常全部陷入阻塞的的情况,即服务发生雪崩的情况。

### 熔断机制

​	熔断机制是服务雪崩的一种有效解决方案。当指定时间窗内的请求失败率达到设定阈值时,系统将通过断路器直接将此请求链路断开。常见的熔断有两种:

* 预熔断
* 即时熔断

## 服务降级简介

​	服务降级是请求发生问题时的一种增强用户体验的方式。

​	现代系统中,当发生服务熔断时,一定会发生服务降级;但发生服务降级,并不一定是发生了服务熔断。

服务降级埋点节点：

![image-20191008170917511](../assets-images/image-20191008170917511.png) 

# Hystrix简介

​	Spring Cloud 是通过 Hystrix 来实现服务熔断与降级的。

## [官网wiki](https://github.com/Netflix/Hystrix/wiki)

![image-20191008163904935](../assets-images/image-20191008163904935.png)

![image-20191008163945917](../assets-images/image-20191008163945917.png)

> 【原文】In a distributed environment, inevitably(不可避免地) some of the many service dependencies will fail. Hystrix is a library that helps you control the interactions(交互) between these distributed services by adding ==latency tolerance==(延迟容忍) and ==fault tolerance logic==(容 错逻辑). Hystrix does this by isolating points of access between the services, stopping ==cascading failures==(级联错误) ==across them==(跨服务), and providing ==fallback options==(回退选项), all of which improve your system’s overall resiliency(弹性). 
>
> 【翻译】在分布式环境中,许多服务依赖中的一些服务发生失败是不可避免的。Hystrix 是一 个库,通过添加延迟容忍和容错逻辑,帮助你控制这些分布式服务之间的交互。Hystrix 通过 隔离服务之间的访问点、停止跨服务的级联故障以及提供回退选项来实现这一点,所有这些 都可以提高系统的整体弹性。 

## 综合说明

​	当 Hystrix 监控到某个服务发生故障后,其不会让该服务的消费者阻塞,或向消费者抛出异常,而是向消费者返回一个符合预期的、可处理的备选响应(FallBack),这样就避免了服务雪崩的发生。

# fallbackMethod 服务降级

​	Hystrix 对于服务降级的实现方式有两种:**fallbackMethod 服务降级**,与 **fallbackFactory 服务降级**。首先来看 fallbackMethod 服务降级。

## 创建消费者工程 04-consumer-fallbackmethod-8080

### 创建工程

复制 02-consumer-8080 工程,并重命名为 04-consumer-fallbackmethod-8080。 该工程的运行说明 hystrix 本身与 feign 是没有关系的。 

### 添加 hystrix 依赖

```xml
<!--hystrix依赖-->
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

### 修改处理器类

```java
 // 指定该方法要使用服务降级，即若当前处理器方法在运行过程中发生异常，
    // 法给客户端正常响应时，就会调用fallbackMethod指定的方法
    @HystrixCommand(fallbackMethod = "getHystrixHandle")
    @GetMapping("/get/{id}")
    public Depart getHandle(@PathVariable("id") int id) {
        String url = "http://localhost:8081/provider/depart/get/" + id;
        return restTemplate.getForObject(url, Depart.class);
    }

    // 服务降级方法，即响应给客户端的备选方案
    public Depart getHystrixHandle(@PathVariable("id") int id) {
        Depart depart = new Depart();
        depart.setId(id);
        depart.setName("no this depart");
        return depart;
    }
```

### 在启动类添加注解@EnableCircuitBreaker

```java
@EnableCircuitBreaker  // 开启熔断器
@SpringBootApplication
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }

}
```

​	这一个类上添加的注解太多了,为了避免这种情况的发生,Spring Cloud 专门定义了一个组合注解`@SpringCloudApplication`,就包含了这三个注解。所以可以用它直接替换掉那三个注解。

```java
// @EnableCircuitBreaker  // 开启熔断器
// @SpringBootApplication
@SpringCloudApplication
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }

}
```

## 创建消费者工程 04-consumer-feign-fallbackmethod-8080

### 创建工程

​	复制 03-consumer-feign-8080 工程,并重命名为 04-consumer-feign-fallbackmethod-8080。该工程是 Hystrix 与 Feign 结合使用。当然,一般情况下都是这样使用的。

### 添加 hystrix 依赖

```xml
<!--hystrix依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

### 修改处理器类

```java
@HystrixCommand(fallbackMethod = "getHystrixHandle")
    @GetMapping("/get/{id}")
    public Depart getHandle(@PathVariable("id") int id) {
        return service.getDepartById(id);
    }

    public Depart getHystrixHandle(@PathVariable("id") int id) {
        Depart depart = new Depart();
        depart.setId(id);
        depart.setName("no this depart2");
        return depart;
    }
```

### 在启动类添加注解@SpringCloudApplication

```java
// 指定Service接口所在的包，开启OpenFeign客户端
@EnableFeignClients(basePackages = "com.abc.consumer.service")
@SpringCloudApplication
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }

}
```

# fallbackFactory 服务降级

# Hystrix 高级属性配置

## 执行隔离策略

1. 类型
2. 修改策略
3. 默认值

## 线程执行超时时限

## 超时中断

## 取消中断

## 线程池相关属性

c

# Dashboard 监控仪表盘

# 服务降级报警机制