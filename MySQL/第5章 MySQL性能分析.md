# 慢查询日志

show variables like '%slow_query%';

show variables like '%long_query_time%';

systemctl restart mysqld

## 性能优化思路

1. 首先需要使用慢查询功能,去获取所有查询时间比较长的SQL语句 

2. 其次使用explain命令去查看有问题的SQL的执行计划 

3. 最后可以使用show profile[s] 查看有问题的SQL的性能使用情况 

4. 优化改造SQL语句(需要对于需求有很好的理解,比如查询某个人最近半年的银行流水,银行只会提供最近一年 

   内的流水,还有的只提供最近半年的查询)

## 慢查询日志介绍

​	数据库查询快慢是影响项目性能的一大因素,对于数据库,我们除了要优化 SQL,更重要的是得先找到需要优化的SQL。

​	MySQL 数据库有一个"慢查询日志"功能,用来记录查询时间超过某个设定的值的SQL ,这将极大程度帮助我们快速定位为症结所在 ,以便对症下药。 

​	至于查询时间的多少才算慢,每个项目、业务都有不同的要求。

​	比如说传统企业的软件允许查询时间高于某个值,但是把这个标准放在互联网项目或者访问量大的网站上,估计就是一个bug,甚至可能升级为一个功能性缺陷。 

​	MySQL的慢查询日志功能,  默认是关闭的，需要手动开启。

## 开启慢查询功能

* 查看是否开启慢查询功能

  ```mysql
  show variables like '%slow_query%';
  
  show variables like '%long_query_time%';
  ```

  参数说明：

  ```mysql
  -- slow_query_log :是否开启慢查询日志,ON 为开启,OFF 为关闭,如果为关闭可以开启。
  
  -- log-slow-queries :旧版(5.6以下版本)MySQL数据库慢查询日志存储路径。可以不设置该参数,系统则会默认给一个缺省的文件host_name-slow.log
  
  -- slow-query-log-file:新版(5.6及以上版本)MySQL数据库慢查询日志存储路径。可以不设置该参数,系统则会默认给一个缺省的文件host_name-slow.log
  
  -- ong_query_time :慢查询阈值,当查询时间多于设定的阈值时,记录日志,单位为秒。
  ```

  

## 分析慢查询日志

### MySQL自带的mysqldumpslow

### 使用mysqlsla工具

### 使用persona-toolkit工具

# 查看执行计划

## 介绍

## 参数说明

## 参考网站

# profile分析语句

## 介绍

## 语句使用

## 开启profile功能

## 示例