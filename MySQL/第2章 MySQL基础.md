# MySQL单机安装

```shell
操作系统:CentOS 7 
MySQL:5.6
```

## MySQL的卸载

### 查看MySQL软件

```shell
rpm -qa|grep mysql
```

### 卸载MySQL

```shell
yum remove -y mysql mysql-libs mysql-common
rm -rf /var/lib/mysql
rm /etc/my.cnf
```

查看是否还有 MySQL 软件,有的话继续删除。
软件卸载完毕后如果需要可以删除 MySQL 的数据库: `/var/lib/mysql`

## 安装MySQL

```shell
wget http://dev.mysql.com/get/mysql-community-release-el6-5.noarch.rpm
rpm -ivh mysql-community-release-el6-5.noarch.rpm
yum install -y mysql-community-server
```

## 配置MySQL

```shell
vim /etc/my.cnf
```

修改内容如下：

```shell
[mysqld]
# MySQL设置大小写不敏感:默认:区分表名的大小写,不区分列名的大小写 # 0:大小写敏感 1:大小写不敏感
lower_case_table_names=1
# 默认字符集
default-character-set=utf8
```

## 启动MySQL

```shell
systemctl start mysqld
```

## 设置root用户密码

例如:为 `root` 账号设置密码为 `111111` :

```shell
/usr/bin/mysqladmin -u root password 'root'
```

## 登录MySQL

* 登录命令：

  ```shell
  #-u:指定数据库用户名 -p:指定数据库密码,记住-u和登录密码之间没有空格
  mysql -uroot -proot
  ```

## MySQL远程连接授权

* 授权命令

  ```shell
  grant 权限 on 数据库对象 to 用户
  ```

* 示例：授予`root`用户对所有数据库对象的全部操作权限:

  ```shell
  mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT
  OPTION;
  ```

* 命令说明

  * `ALL PRIVILEGES` :表示授予所有的权限,此处可以指定具体的授权权限。
  * `*.*` :表示所有库中的所有表
  * `'root'@'%'` : myuser是数据库的用户名,%表示是任意ip地址,可以指定具体ip地址。
  * `IDENTIFIED BY 'mypassword'` :mypassword是数据库的密码。

## 关闭Linux防火墙

```shell
systemctl stop firewalld(默认)
systemctl disable firewalld.service(设置开启不启动)
```

# DDL语句

## 数据库操作：database

### 创建数据库

```mysql
create database 数据库名;
create database 数据库名 character set 字符集;
```

### 查看数据库

* 查看数据库服务器中的所有的数据库:

  ```mysql
  show databases;
  ```

* 查看某个数据库的定义的信息:

  ```mysql
  show create database 数据库名;
  ```

### 删除数据库（慎用）

```mysql
drop database 数据库名称;
```

### 其他数据库操作命令

* 切换数据库:

  ```mysql
  use 数据库名;
  ```

* 查看正在使用的数据库:

  ```mysql
  select database();
  ```

## 表操纵：table

### 字段类型

* 常用的类型有：

  * 数字型:int
  * 浮点型:double 
  * 字符型:varchar(可变长字符串) 
  * 日期类型:date(只有年月日,没有时分秒) 
    datetime(年月日,时分秒) 
  * boolean类型:不支持,一般使用tinyint替代(值为0和1) 

  ![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0ay9hdah9j30r00s245o.jpg)

  ![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0ayce3fgvj30r20cwn04.jpg)

### 创建表

```mysql
create table 表名(
字段名 类型(长度) 约束, 字段名 类型(长度) 约束
);
```

* 单表约束

  ```sql
  - 主键约束:primary key 
  - 唯一约束:unique
  - 非空约束:not null
  ```

* 注意：

  ```mysql
  主键约束 = 唯一约束 + 非空约束
  ```

### 查看表

* 查看数据库中的所有表:

  ```mysql
  show tables;
  ```

* 查看表结构:

  ```mysql
  desc 表名;
  ```

### 删除表

```sql
drop table 表名;
```

### 修改表

```sql
alter table 表名 add 列名 类型(长度) 约束; --修改表添加列.
alter table 表名 modify 列名 类型(长度) 约束; --修改表修改列的类型长度及约束. alter table 表名 change 旧列名 新列名 类型(长度) 约束; --修改表修改列名.
alter table 表名 drop 列名;
rename table 表名 to 新表名;
alter table 表名 character set 字符集;
--修改表删除列. --修改表名 --修改表的字符集
```

# DML语句

## 插入记录：insert

* 语法

  ```sql
  insert into 表 (列名1,列名2,列名3..) values (值1,值2,值3..); -- 向表中插入某些列 
  insert into 表 values (值1,值2,值3..); --向表中插入所有列
  insert into 表 (列名1,列名2,列名3..) values select (列名1,列名2,列名3..) from 表; 
  insert into 表 values select * from 表;
  ```

* 注意

  1. 列名数与values后面的值的个数相等 
  2. 列的顺序与插入的值得顺序一致 
  3. 列名的类型与插入的值要一致. 
  4. 插入值得时候不能超过最大长度. 
  5. 值如果是字符串或者日期需要加引号’’ (一般是单引号 )

* 例如：

  ```sql
  INSERT INTO sort(sid,sname) VALUES('s001', '电器'); 
  INSERT INTO sort(sid,sname) VALUES('s002', '服饰'); 
  INSERT INTO sort VALUES('s003', '化妆品');
  INSERT INTO sort VALUES('s004','书籍');
  ```

## 更新记录：update

* 语法

  ```sql
  update 表名 set 字段名=值,字段名=值;
  update 表名 set 字段名=值,字段名=值 where 条件;
  ```

* 注意

  * 列名的类型与修改的值要一致.
  * 修改值得时候不能超过最大长度.
  * 值如果是字符串或者日期需要加‘ ’.

## 删除记录：delete

* 语法

  ```sql
  delete from 表名 [where 条件];
  ```

* 面试题

  ```sql
  --删除表中所有记录使用【delete from 表名】,还是用【truncate table 表名】?
  --删除方式:
  -- delete :一条一条删除,不清空auto_increment记录数。
  -- truncate :直接将表删除,重新建表,auto_increment将置为零,从新开始。
  ```

# DQL语句

## 准备工作

```mysql
CREATE TABLE products (
pid INT PRIMARY KEY AUTO_INCREMENT, # 自增加 AUTO_INCREMENT pname VARCHAR(20),#商品名称
price DOUBLE, #商品价格
pdate DATE, # 日期
sid VARCHAR(20) #分类ID
);

INSERT INTO products VALUES(NULL,'泰国大榴莲', 98, NULL, 's001'); 
INSERT INTO products VALUES(NULL,'新疆大枣', 38, NULL, 's002'); 
INSERT INTO products VALUES(NULL,'新疆切糕', 68, NULL, 's001'); 
INSERT INTO products VALUES(NULL,'十三香', 10, NULL, 's002'); 
INSERT INTO products VALUES(NULL,'老干妈', 20, NULL, 's002');
```

## DQL语法顺序

完整DQL语法顺序

```sql
SELECT DISTINCT
    < select_list >
FROM
    < left_table > < join_type >
JOIN < right_table > ON < join_condition >
WHERE
    < where_condition >
GROUP BY
    < group_by_list >
HAVING
    < having_condition >
ORDER BY
    < order_by_condition >
LIMIT < limit_number >
```

### 简单查询



### 条件查询

### 排序

### 聚合函数（组函数）

### 分组

## 分页查询

## 子查询

## 其他查询语句



## 

# SQL解析顺序

## 准备工作

## 1.FROM

## 2.ON过滤

## 3.OUTER JOIN添加外部列

## 4.WHERE

## 5.GROUP BY

## 6.HAVING

## 7.SELECT

## 8.DISTINCT

## 9.ORDER BY

## 10.LIMT（MySQL特有）

## 解析顺序总结

### 图示

### 流程分析

### WHERE条件解析顺序

# 多表之间的关系

## 表与表之间的关系

## 外键

## 一对一关系（了解）

## 一对多关系

## 多对多关系



# 多表关联查询

## 交叉连接

## 内连接

## 外连接

# MySQL锁

## MySQL锁介绍

## MySQL表级锁

### 表级锁介绍

### 表锁介绍

### 表锁演示

#### 环境准备

#### 读锁演示

#### 写锁演示

### 元数据锁介绍

### 元数据锁演示

## MySQL行级锁

### 行级锁介绍

### InnoDB行锁演示

#### 创建表及索引

#### 行锁定基本演示

#### 无索引升级为表锁演示

#### 间隙锁带来的插入问题演示

#### 使用共同索引不同数据的阻塞示例

#### 死锁演示

# MySQL事务

## 事务介绍

## 事务开启

## 事务并发问题

## 事务隔离级别

# MySQL索引

## 索引介绍

## 索引的分类

## 索引的使用