# 第1章 MyBatis整体架构

## 1.1 MyBatis与ORM

​	**ORM**是**Object**与**Relation**之间映射. 包括**Object->Relation**和**Relation->Object**两方面.**Hibernate**是一个完整**ORM**框架. 而**MyBatis**完成的是**Relation->Object**.这一点可以从其[官方网站](https://mvnrepository.com/artifact/org.mybatis)看到。

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g05z52h1e4j319t0bwtdd.jpg)

​	**MyBatis**并不刻意于完成**ORM**(对象映射)的完整概念. 而是旨在让开发人员使用更简单、更方便的方式完成数据库操作功能. 这对于应用系统来说也是最实用的

## 1.2 MyBatis架构

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g05z6lsuohj312i0u0b29.jpg)

### 1.2.1 接口层

​	接口层是**MyBatis**提供给开发人员的一套**API**.主要使用**SqlSession**接口.通过**SqlSession**接口和**Mapper**接口.开发人员,可以通知**MyBatis**框架调用那一条**SQL**命令以及**SQL**命令关联参数.

* **SqlSession**接口使用方式：
  ![](https://ws4.sinaimg.cn/large/006tKfTcgy1g05zd88lioj31d80a9jwk.jpg)
* **Mapper**接口使用方式：
  ![](https://ws2.sinaimg.cn/large/006tKfTcgy1g05zdus0gbj31e3082wh2.jpg)

### 1.2.2 数据处理层

​	数据处理层是**MyBatis**框架内部实现。来完成对数据库具体操作。主要负责：

1. 参数与**SQL**命令绑定；
2. **SQL**命令发送方式；
3. 查询结果类型转换；

### 1.2.3 支撑层

​	支撑层用来完成**MyBatis**与数据库基本连接方式以及**SQL**命令与配置文件对应。主要负责：

1. **MyBatis**与数据库连接方式管理；
2. **MyBatis**对事务管理方式；
3. **SQL**命令与**XML**配置对应；
4. **MyBatis**查询缓存管理

## 1.3 MyBatis调用流程

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g05zrwkfzqj30u012ogyf.jpg)

1. **`SqlSession`**：接收开发人员提供**Statement Id** 和参数.并返回操作结果；
2. **`Executor`**：**MyBatis**执行器，是**MyBatis** 调度的核心，负责SQL语句的生成和查询缓存的维护；
3. **`StatementHandler`**：封装了**JDBC Statement**操作，负责对**JDBC statement** 的操作，如设置参数、将**Statement**结果集转换成**List**集合；
4. **`ParameterHandler`**：负责对用户传递的参数转换成**JDBC Statement** 所需要的参数；
5. **`ResultSetHandler`**：负责将**JDBC**返回的**ResultSet**结果集对象转换成**List**类型的集合；
6. **`TypeHandler`**：负责**java**数据类型和**jdbc**数据类型之间的映射和转换；
7. **`MappedStatement`**：维护了一条`<select|update|delete|insert>`节点的封装；
8. **`SqlSource`**：负责根据用户传递的**parameterObject**，动态地生成**SQL**语句，将信息封装到**BoundSql**对象中，并返回**BoundSql**表示动态生成的**SQL**语句以及相应的参数信息；
9. **`Configuration`**：**MyBatis**所有的配置信息都维持在**Configuration**对象之中

## 1.4 MyBatis框架使用方式

### 1.4.1 基于XML配置文件

SQL命令声明在XML配置文件中

### 1.4.2 基于注解方式

SQL命令声明在注解中

# 第2章 SessionFactory机制原理

## 2.1 SqlSessionFactory基本介绍

​	对于任何框架而言，在使用前都要进行一系列的初始化，**MyBatis**也不例外.**SqlSessionFactory**是**MyBatis**框架中的一个接口,主要负责**MyBatis**框架初始化操作以及为开发人员提供**SqlSession**对象.

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g060gi7efuj31c10hedmj.jpg)

**SqlSessionFactory**有两个实现类：

1. **SqlSessionManager**类(被抛弃)
2. **DefaultSqlSessionFactory**类
   ![](https://ws4.sinaimg.cn/large/006tKfTcgy1g060ijb0imj31d30po7dz.jpg)

## 2.2 SqlSessionFactory基本执行流程

1. 调用 **SqlSessionFactoryBuilder** 对象的 `build(inputStream)` 方法；
2. **SqlSessionFactoryBuilder** 会根据输入流 **inputStream** 等信息创建**XMLConfigBuilder** 对象 ;
3. **SqlSessionFactoryBuilder** 调用 **XMLConfigBuilder** 对象的 `parse()` 方法；
4.  **XMLConfigBuilder** 对象返回 **Configuration** 对象；
5. **SqlSessionFactoryBuilder**创建一个**DefaultSessionFactory** 对象,并将**Configuration**对象作为参数传给**DefaultSessionFactory**对象；
6. **SqlSessionFactoryBuilder** 返回 **DefaultSessionFactory** 对象给 **Client** ，供 **Client**使用。**Client**可以使用**DefaultSessionFactory**对象创建需要的**SqlSession**.

上述的初始化过程中，涉及到了以下几个对象：

* **SqlSessionFactoryBuilder** ： **SqlSessionFactory**的构造器，用于创建**SqlSessionFactory**，采用了**Builder**设计模式
* **Configuration** ：该对象是`mybatis-config.xml`文件中所有**mybatis**配置信息
* **SqlSessionFactory**：**SqlSession**工厂类，以工厂形式创建**SqlSession**对象，采用了**Factory**工厂设计模式
* **XmlConfigParser** ：负责将`mybatis-config.xml`配置文件解析成**Configuration**对象，供**SqlSessonFactoryBuilder**使用，创建**SqlSessionFactory**

## 2.3 SqlSessionFactory涉及的设计模式——Builder（建造者模式）

### 2.3.1 什么是Builder设计模式

​	**Builder**模式,称为[建造者设计模式].使用多个简单的对象一步一步构建一个复杂的对象.这种模式属于[创建型模式].目前它是创建对象的最佳模式.

### 2.3.2 SqlSessionFactoryBuilder与SqlSessionFactory之间关系

​	**SqlSessionFactoryBuilder**是**Builder**模式中建造者,负责**SqlSessionFactory**对象的创建以及

**SqlSessionFactroy**对象内部所需要内容的组装.

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g060wshopvj31d60jdahb.jpg)

# 第3章 Configuration介绍

## 3.1 Configuration介绍

​	**MyBatis**框架支持开发人员通过配置文件与其进行交流.在配置文件所配置的信息,在框架运行时,会被**XMLConfigBuilder**解析并存储在一个**Configuration**对象中.**Configuration**对象会被作为参数传送给**DeFaultSqlSessionFactory**.而**DeFaultSqlSessionFactory**根据**Configuration**对象信息为**Client**创建对应特征的**SqlSession**对象。

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0612hwbmvj31d60pwh3e.jpg)

## 3.2 MyBatis配置文件回顾

# 第4章 SqlSession中四大神器之Executor(执行器)

## 4.1 Executor简介

​	每一个**SqlSession**对象都会拥有一个**Executor**(执行器对象);这个执行对象负责[增删改查]的具体操作.我们可以简单的将它理解为JDBC中**Statement**的封装版.

## 4.2 Executor继承结构

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g061ijlp9vj31fw0ron3e.jpg)

* `Executor`是一个接口;主要有两个实现类.分别是[`BaseExecutor`]和[`CachingExecutor`].
* [`BaseExecutor`]是一个抽象类.这种通过抽象类实现接口的方式是[适配器设计模式]的体现。主要用于方便次一级子类对接口中方法的实现.
* [`BaseExecutor`]主要有三个实现类[`SimpleExecutor`],[ `ReuseExecutor`],[ `BatchExecutor`]
  * [`SimpleExecutor`]被称为[**简单执行器**],是**MyBatis**中默认使用的执行器. 每执行一次`update`或`select`，就开启一个**Statement**对象，用完立刻关闭**Statement**对象。（可以是**Statement**或**PrepareStatement**对象）
  * [ `ReuseExecutor`]被称为[**可重用执行器**].这里的重用指的是重复使用**Statement**. 它会在内部利用一个**Map**把创建的**Statement**都缓存起来，每次在执行一条**SQL**语句时，它都会去判断之前是否存在基于该**SQL**缓存的**Statement**对象，存在而且之前缓存的**Statement**对象对应的**Connection**还没有关闭的时候就继续用之前的**Statement**对象，否则将创建一个新的**Statement**对象，并将其缓存起来。因为每一个新的**SqlSession**都有一个新的**Executor**对象，所以我们缓存在**ReuseExecutor**上的**Statement**的作用域是同一个SqlSession。
  * [ `BatchExecutor`]称为[批处理执行器].用于将多个sql语句一次性输送到数据库执行.
* [`CachingExecutor`]称为[**缓存执行器**]. 先从缓存中获取查询结果，存在就返回，不存在，再委托给**Executor delegate**去数据库取，**delegate**可以是上面任一的**SimpleExecutor**、**ReuseExecutor**、**BatchExecutor**。代码如下：

## 4.3 Executor对象创建

​	执行器对象是由**Coniguration**对象负责创建的.**Configuration**对象会根据得到[`ExecutorType`]创建对应的**Excecutor**对象,并把这个**Excecutor**对象传给**SqlSession**对象

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g061vk1xo0j31d80ig7f3.jpg)

## 4.4 ExecutorType的选择

​	**ExecutorType**来决定**Configuration**对象创建何种类型的执行器.它的赋值可以通过两个地方进行赋值:

* 首先.可以通过`<settings>`标签来设置当前工程中所有**SqlSession**对象使用的默认**Executour**
  ![](https://ws1.sinaimg.cn/large/006tKfTcgy1g061zp9fqbj31d60csjxk.jpg)
* 也可以通过**SqlSessoinFactory**中`openSession`方法来指定具体的**SqlSession**使用的执行器
  ![](https://ws1.sinaimg.cn/large/006tKfTcgy1g061z5lfeij31d608vaev.jpg)

# 第5章 SqlSession中四大神器之StatementHandler

## 5.1 StatementHandler简介

​	**StatementHandler**是四大神器中最重要的一个对象,负责操作**Statement**与数据库进行交流.在工作时

还会使用**ParameterHandler**进行参数配置,使用**ResultHandler**将查询结果与实体类对象进行绑定.

首先看一下StatementHandler接口定义：

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g06285ji57j31bv0u0458.jpg)

在**StatementHandler**接口中有四种重要的方法.分别是`prepare`,`parameterize`,`batch`,`update`.

* `prepare`：用于具体创建一个**Statement**对象或则**preparedStatement**对象

* `parameterize`：用于初始化**Statement**及对**Sql**中占位符进行赋值.

* `update`：用于通知**Statement**将[`insert`,`update`,`delete`]推送到数据库

* `query`：用于通知**Statement**将[`select`]推送到数据库并返回对应查询结果.

## 5.2 StatementHandler继承结构

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g062bvnf5mj31d80sz4kw.jpg)

从上图可以看到`StatementHandler`接口下有两个直接实现类[`BaseStatementHandler`]和[`RoutingStatementHandler`]

* [`RoutingStatementHandler`]是一个具体实现类.在这个类中并没有对Statement对象进行具体使用.只是根据得到**Executor**类型,决定创建何种类型**StatementHandler**对象.==在**MyBatis**工作时,使用的`StatementHandler`接口对象实际上就是**RoutingStatementHandler**对象==.我们可以简单理解为:

  ```java
  StatementHandler statmentHandler = new RountingStatementHandler();
  ```

* [`BaseStatementHandler`]是`StatementHandler`接口的另一个实现类.本身是一个抽象类.用于简化`StatementHandler`接口实现的难度,属于**适配器设计模式**体现.它有三个实现类：**`SimpleStatementHandler`**,**`PreparedStatementHandler`**,**`CallableStatementHandler`**

* 在`RountingStatementHandler`创建时,就跟根据接收的**Executor**类型来创建这个三个类型对象的
  * `SimpleStatementHandler`：管理**Statement**对象向数据库中推送不需要预编译的**SQL**语句
  * `PreparedStatementHandler`：管理**PreparedStatementHandler**对象向数据库推送预编译的**SQL**语句
  * `CallableStatementHandler`：管理**CallableStatement**对象调用数据库中构造函数

## 5.3 StatementHandler对象创建

* **StatementHandler**对象是在**SqlSession**对象接收到操作命令时,由**Configuraion**中**newStatementHandler**方法负责调用的.
  ![](https://ws3.sinaimg.cn/large/006tKfTcgy1g062tgx3ewj31d509wtdy.jpg)
* **RoutingStatementHandler**构造方法,将会根据**Executor**的类型决定创建**SimpleStatementHandler**,**PreparedStatementHandler**,**CallableStatementHandler**实例对象.
  ![](https://ws3.sinaimg.cn/large/006tKfTcgy1g062tvr5q8j31b40mh7dq.jpg)

## 5.4 StatementHandler接口方法介绍

### 5.4.1 prepare方法

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g062xw09wjj31d60nn0zi.jpg)

*  `prepare`方法用于创建一个(**Statement or PreparedStatement or CallableStatement**)对象,并设置**Statement**对象的最大工作时间和一次性读取的最大数据量.然后将生成的**Statement**对象返回.
* `prepare`方法只在**BaseStatementHandler**被实现.在其三个子类中没有被重写.用于三个子类调用获得对应的**Statement**接口对象.
* `prepare`方法依靠`instantiateStatement(connection)`方法来返回具体**Statement**接口对象.这个方法是**BaseStatementHandler**中定义的抽象方法,由三个子类来具体实现.
  * **SimpleHandler**中的[`instantiateStatement`] 方法
    ![](https://ws4.sinaimg.cn/large/006tKfTcgy1g063211p8hj31d80feag4.jpg)
  * **PreparedStatementHandler**中的[`instantiateStatement`] 方法
    ![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0632gbsvoj31d80kjk0w.jpg)
  * **CallableStatementHandler**中的[`instantiateStatement`] 方法
    ![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0632tz6toj31d50h2agi.jpg)

### 5.4.2 parameterize方法

​	主要为**PreparedStatement**和**CallableStatement**传参.因此只在**PreparedStatementHandler**和**CallableStatementHandler**中被重写.

* **PreparedStatementHandler**中的`parameterize`
  ![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0635mms2qj31d0087n0e.jpg)

* **CallableStatementHandler**中的`parameterize`
  ![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0635y3saej31d809o795.jpg)

在这两个方法中,可以看到都是[**ParameterHandler**]对象进行参数赋值的.

### 5.4.3 query方法

​	输送查询语句,并将查询结果转换对应的实体类对象

*  **SimpleStatementHandler** 中的 `query` 方法
  ![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0638j5x65j319q09wdje.jpg)
* **PreparedStatementHandler**中的 `query` 方法
  ![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0638tfk0pj31am0bbaec.jpg)
* **CallableStatementHandler**中的 `query` 方法
  ![](https://ws2.sinaimg.cn/large/006tKfTcgy1g06392t1j7j31ba0brjwd.jpg)

可以看到在得到查询结果后,都是使用[**ResultSetHandler**]对结果进行转换.

### 5.4.4 update方法

​	输送[`insert`,`update`,`delete`]语句并返回处理数据行

* **SimpleStatementHandler**中的`update`方法
  ![](https://ws1.sinaimg.cn/large/006tKfTcgy1g063ef94csj31d60o0wrt.jpg)
* **PreparedStatementHandler**中的`update`方法
  ![](https://ws2.sinaimg.cn/large/006tKfTcgy1g063erhoipj31d80f6qbi.jpg)
* **CallableStatementHandler**中的`update`方法
  ![](https://ws3.sinaimg.cn/large/006tKfTcgy1g063f52z03j31d80dvqba.jpg)​        

# 第6章 SqlSession中四大神器之ParameterHandler

## 6.1 ParameterHandler简介

​	参数处理器,负责为**PreparedStatement**的**sql**语句参数动态赋值
![](https://ws3.sinaimg.cn/large/006tKfTcgy1g063jj7e1sj31d60k443r.jpg)

 这个接口中只有两个方法：

*  `getParameterObject` 方法,用于读取参数.

* `setParameters`用于对**PreparedStatement**的参数赋值.

## 6.2 ParameterHandler继承结构

只有一个实现类：**DefaultParameterHandler**

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g063jv47wtj31d60eb46x.jpg)

## 6.3 ParameterHandler对象创建

​	参数处理器对象是在创建**StatementHandler**对象同时被创建的.由**Configuration**对象负责创建.

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g063lxq89qj31d60hnth5.jpg)

创建时传入三个对象：执行SQL对应的配置信息**MappedStatement**、参数对象**Object**，SQL的**BoundSql**。

> 注：一个**BoundSql**对象，代表了一次sql语句的实际执行，而**SqlSource**对象的责任，就是根据传入的参数对象，动态计算出这个**BoundSql**，也就是说**Mapper**文件中的节点的计算，是由**SqlSource**对象完成的。**SqlSource**最常用的实现类是**DynamicSqlSource**

## 6.4 ParameterHandler中的参数从何而来

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g063odm6hlj31a10cnn1s.jpg)

上述命令的实参**`10`**是如何添加到对应的**SQL**语句中的.

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g063p3dow7j319z09tae3.jpg)

在**mybatis**中,使用动态代理模式.当`dao.findByDeptNo(10)`将要执行时;会被**JVM**进行拦截交给**mybatis**中的代理实现类**MapperProxy**的`invoke`方法中
![](https://ws3.sinaimg.cn/large/006tKfTcgy1g063q81un4j31d80i546x.jpg)

然后再一步步交给**ParameterHandler**中`setParameter`方法,将参数交给对应占位符

## 6.5 ParameterHandler中

```java
public void setParameters(PreparedStatement ps) {
		ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());
		// 获取要设置的参数映射信息  parameterMappings是对#{}里面给出的参数信息的封装，即这个SQL是个参数化SQL(SQL语句中带占位符?)时，会设置参数
		List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
		//如果是参数化SQL，便要设置参数
		if (parameterMappings != null) {
			for (int i = 0; i < parameterMappings.size(); i++) {
				ParameterMapping parameterMapping = parameterMappings.get(i);
				// 只处理入参
				//如果参数的类型不是OUT类型，CallableStatementHandler会用到，因为存储过程才存在输出参数，所以当参数不是输出参数的时候，就需要设置
				if (parameterMapping.getMode() != ParameterMode.OUT) {
					Object value;
					// 获取属性名称
					//在本次查询中，对应的不是一个Bean，如果是Bean则通过属性的反射来得到value，对应的是一个additionalParameter Map对象，
					String propertyName = parameterMapping.getProperty();
					//如果propertyName是additionalParameter中的key
					if (boundSql.hasAdditionalParameter(propertyName)) { // issue #448 ask first for additional params
						//通过key来得到Map中的value
						value = boundSql.getAdditionalParameter(propertyName);
					} //如果不是additionalParameter中的key，而且传入参数是null，那value就为null
					else if (parameterObject == null) {
						value = null;
					} //如果在typeHandlerRegistry中已经注册了这个参数的class对象，即它是primitive类型或者是String
					else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
						value = parameterObject;
					} //否则就是Map（Method穿进来的原始参数是List类型或者是Array类型时，这时已经被封装成了Map类型了）
					else {
						MetaObject metaObject = configuration.newMetaObject(parameterObject);
						value = metaObject.getValue(propertyName);
					}
					// 获取每个参数的类型处理器，去设置入参和获取返回值
					TypeHandler typeHandler = parameterMapping.getTypeHandler();
					// 获取每个参数的JdbcType，得到相应parameterMappings的jdbcType，如果没有在#{}中显示指定jdbcType，则为null
					JdbcType jdbcType = parameterMapping.getJdbcType();
					if (value == null && jdbcType == null) {
						jdbcType = configuration.getJdbcTypeForNull();
					}
					try {
						// 给PreparedStatement设置参数，这行实现了具体的ps.set***
						typeHandler.setParameter(ps, i + 1, value, jdbcType);
					} catch (TypeException e) {
						throw new TypeException(
								"Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
					} catch (SQLException e) {
						throw new TypeException(
								"Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
					}
				}
			}
		}
	}
```

# 第7章 SqlSession中四大神器之ResultSetHandler

## 7.1 ResultSetHandler简介

ResultSetHandler接口主要负责两件事：

1. 处理Statement执行后产生的结果集，生成结果列表；
2. 处理存储过程执行后的输出参数； 

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g072g1kczoj31d80hn7a9.jpg)

## 7.2 ResultSetHandler继承结构

只有一个实现类**DefaultResultSetHandler**

# 第8章 MyBatis事务管理

## 8.1 什么是事务

​	事务指的是一个最小的业务单位.在实际工作中即使最小的一个业务也往往会涉及到对数据库的多次操作,这里增加出错的几率.为了减少出错时维护负担.一般要求将同一个事务中的**SQL**语句保存到数据库中的同一**Transaction**对象中,原因是**Transaction**具有**一致性**的特征,也就是说**事务中如果有任何一条sql语句运行失败,那么这个事务中所有的SQL语句都会被判定为无效SQL**,这对维护是有很大帮助的.

## 8.2 JDBC中事务手动管理代码

```java
con.setAutoCommit(false);
//此处命令通知数据库,从此刻开始从当前Connection通道推送而来的SQL语句属于同一个业务中,这些SQL语句在数据库中应该保存到同一个Transaction中.这个Transaction的行为(commit,rollback)由当前Connection管理.
try{
    //推送sql语句命令……..;
    con.commit();//通知Transaction提交.
}catch(SQLException ex){
  con.rollback();//通知Transaction回滚.
}

```

## 8.3 MyBatis对事务管理策略

​	**MyBatis**框架就是对**JDBC**的商业封装,因此也对上述代码(8.2)进行封装.在**MyBatis**中对于上述代码管理有两种方式.

### 8.3.1 亲自实现JDBC管理方式

​	如果当前**MyBatis**是单独运行,没有被其它框架进行管理.此时调用的是MyBatis内部对上述代码实现

### 8.3.2 委托实现JDBC管理方式

​        如果在运行时,**MyBatis**收到了其它框架的管理,比如**Spring**.此时可以将对**Transaction**的管理交由对应**Spring**框架进行管理实现.

开发人员,可以在**MyBatis**中核心配置文件,指定采用管理方式

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0738jhfn7j319p0u0aoo.jpg)

## 8.4 MyBatis中Transaction接口

**Transactions**接口中对两种事务管理方式,进行行为约束

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g073aoaqcqj31d60j07bd.jpg)

Transaction接口中有四个方法,来描述对事务管理的行为.

* `getConnection`方法: 
  **JDBC**中事务手动管理,需要依靠**Connection**对象.`getConnection`方法约束取得**Connection**对象方式.一般来说,要么是手动**new**一个**Connection**.要么就是从数据库连池获得.
* `commit`方法:
  设置在何种情况下,来执行`con.commit()`这个命令
* `rollback`方法
  设置在何种情况下,来执行`con.rollback`这个命令.
* `close`方法
  设置在业务执行完毕后,如何处理**Connection**对象.一般有两种形式：一是将这个**Connection**对象销毁；二是将**Connection**返回到数据库连接池中.
* `getTimeout`方法
  在**Connection**向数据库索要一个**transaction**对象时,会有一个最大等待时间.`getTimeout`方法返回这个最大的等待时间.

## 8.5 Transaction接口的实现类

​	**MyBatis**对于**Transaction**接口中提供了两个实现类.对应两种transaction管理方案

1.  实现类: **JdbcTransaction**
   当配置文件 `<transactionManager type="JDBC"/>`中时,意味着由**MyBatis**提供**transaction**管理代码的实现.此时就会调用**JdbcTransaction**;
2. 实现类: **ManagedTransaction**
   当配置文件中`<transactionManager type="MANAGED"/>`时,意味着数据库**transaction**的管理实现交由其它框架来负责.此时调用**ManagedTransaction**.我们可以在这个类中看到它的`commit`方法和`rollback`方法没有具体实现.

## 8.6 JdbcTransaction类中的方法介绍

### 8.6.1 getConnection方法

返回**Connection**对象

```java
@Override
  public Connection getConnection() throws SQLException {
    if (connection == null) {
      openConnection();
    }
    return connection;
  }
```

```java
protected void openConnection() throws SQLException {
    if (log.isDebugEnabled()) {
      log.debug("Opening JDBC Connection");
    }
    connection = dataSource.getConnection();
    if (level != null) {
      connection.setTransactionIsolation(level.getLevel());
    }
    setDesiredAutoCommit(autoCommmit);
  }
```

 **`openConnection`**方法中,做了三件事

1. 从数据库连接池得到了一个**Connection**
2. 设置数据库的事务隔离级别
3. 调用了`con.setAutoCommit(false)`;在`setDesireAutoCommit(autoCommit)`

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g073wf6mayj31d60gmjz0.jpg)

### 8.6.2 commit方法

```java
@Override
  public void commit() throws SQLException {
    if (connection != null && !connection.getAutoCommit()) {
      if (log.isDebugEnabled()) {
        log.debug("Committing JDBC Connection [" + connection + "]");
      }
      connection.commit();
    }
  }
```

### 8.6.3 rollback方法

```java
 @Override
  public void rollback() throws SQLException {
    if (connection != null && !connection.getAutoCommit()) {
      if (log.isDebugEnabled()) {
        log.debug("Rolling back JDBC Connection [" + connection + "]");
      }
      connection.rollback();
    }
  }
```

### 8.6.4 close方法

```java
@Override
  public void close() throws SQLException {
    if (connection != null) {
      resetAutoCommit();
      if (log.isDebugEnabled()) {
        log.debug("Closing JDBC Connection [" + connection + "]");
      }
      connection.close();
    }
  }
```

在`close`方法中,我们可以看到在**connection**被回收到数据库连接池之前.执行了一个`resetAutoCommit`方法.

```java
protected void resetAutoCommit() {
    try {
      if (!connection.getAutoCommit()) {
        // MyBatis does not call commit/rollback on a connection if just selects were performed.
        // Some databases start transactions with select statements
        // and they mandate a commit/rollback before closing the connection.
        // A workaround is setting the autocommit to true before closing the connection.
        // Sybase throws an exception here.
        if (log.isDebugEnabled()) {
          log.debug("Resetting autocommit to true on JDBC Connection [" + connection + "]");
        }
        connection.setAutoCommit(true);
      }
    } catch (SQLException e) {
      if (log.isDebugEnabled()) {
        log.debug("Error resetting autocommit to true "
          + "before closing the connection.  Cause: " + e);
      }
    }
  }
```

### 8.6.5 getTimeout方法

```java
@Override
  public Integer getTimeout() throws SQLException {
    return null;
  }
```

# 第9章 乐观锁实现

## 9.1 什么是乐观锁

​	就是一种乐观的状态.认为当前表大多数时候以查询为主,只有在极少的情况下会进行数据更新.所以在乐观锁的世界中,只会考虑多个用户同时提出数据更新时产生数据冲突问题,而不会考虑读数据与写数据之间的冲突.

​	此外乐观锁只是一种策略.不是真的锁.因此在乐观锁的情况下,永远不存在”死锁”的问题.

##  9.2 MyBatis中乐观锁的实现

### 9.2.1 数据版本(version)

​	version就是表中的一个字段,用来记录当前数据行更新的次数.在更新之前用户需要得到当前数据的version值,在更新时比对此时数据的version值与得到version值是否一致。一致就说明此数据可以被更新.不一致就说明这条数据状态已经与之前状态不一致了,所以要结束本次更新,重新获得这条数据,再决定如何更新.

### 9.2.2 时间戳(timestamp)

# 第10章 MyBatis的一级缓冲与二级缓存

## 10.1 缓存介绍

​      缓存就是内存中的一个对象,用于对数据库查询结果的保存.好处减少与数据库交互次数提高响应速度.

## 10.2 什么是会话

​	会话就是一次完整的交流.在一次完整交流过程中往往包含多次请求响应.而发送请求都是同一个户.SqlSession就是在用户与数据库进行一次会话过程中使用的接口.

## 10.3 MyBatis缓存分类

1. 一级缓存:
   也称为[**本地缓存**],用于保存用户在一次会话过程中查询的结果.用户一次会话中只能使用一个**SqlSession**.**一级缓存是自动开启,不允许关闭**
2. 二级缓存:
   也称为[**全局缓存**],是针对某一个表的查询结果存储.可以共享给所有针对这张表查询的用户,也就是说可以在多个**SqlSession**之间共享.

## 10.4 一级缓冲失效原因

1. 用户在一次会话过程中使用了多个**SqlSession**
2. 用户在会话中使用同一个**SqlSession**,但是查询条件发生了变化
3. 用户在会话中使用同一个**SqlSession**,但是清理了缓存
4. 用户在会话中使用同一个**SqlSession**,但是在两次查询之间对数据库进行了修改

## 10.5 二级缓存开启设置

1.    需要在**MyBatis**核心配置文件,通过`settings`标签开发二级缓存.
  ![](https://ws4.sinaimg.cn/large/006tKfTcgy1g074m0lu7cj315u08lq50.jpg)
2.    在对应Mapper文件中添加`<cache></cache>`
  ![](https://ws4.sinaimg.cn/large/006tKfTcgy1g074mj88akj31620fwgs8.jpg)
3.    设置`<cache>`标签属性
  ![](https://ws3.sinaimg.cn/large/006tKfTcgy1g074n79gzkj31d80sgtty.jpg)
4.    设置对应的实体类实现序列化接口
  ![](https://ws3.sinaimg.cn/large/006tKfTcgy1g074nm1jh6j31ca0agjt3.jpg)



