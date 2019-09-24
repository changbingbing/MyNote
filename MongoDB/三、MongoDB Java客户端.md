# [MongoDB Java](https://www.runoob.com/mongodb/mongodb-java.html)

* marphia:基于mongodb的ORM框架
* spring-data-mongodb:spring提供的mongodb客户端
* mongodb-java-driver:官方驱动包

# mongoldb-java-driver

## POM依赖

```xml
<dependency>
  <groupId>org.mongodb</groupId>
  <artifactId>mongo-java-driver</artifactId>
  <version>3.10.1</version>
</dependency>
```

## 测试代码

```java
package com.kkb.mongodb.official.demo;

import java.util.ArrayList;
import java.util.List;

import org.bson.Document;
import org.junit.Before;
import org.junit.Test;

import com.mongodb.MongoClient;
import com.mongodb.client.FindIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import com.mongodb.client.MongoIterable;
import com.mongodb.client.model.Filters;

public class MongodbDemo {
	
	private MongoClient client;

	@Before
	public void init() {
		client = new MongoClient("192.168.10.135",27017);
	}
	
	@Test
	public void connectDB() {
		MongoDatabase database = client.getDatabase("kkb");
	}
	@Test
	public void createCollection() {
		MongoDatabase database = client.getDatabase("kkb");
		database.createCollection("mycollection2");
	}
	@Test
	public void getCollection() {
		MongoDatabase database = client.getDatabase("kkb");
		MongoCollection<Document> collection = database.getCollection("mycollection");
		MongoIterable<String> collectionNames = database.listCollectionNames();
		for (String string : collectionNames) {
			System.out.println("collection name : "+ string);
		}
	}
	@Test
	public void insertDocument() {
		MongoDatabase database = client.getDatabase("kkb");
		MongoCollection<Document> collection = database.getCollection("mycollection2");
		Document document = new Document("name", "James");
		document.append("age", 34);
		document.append("sex", "男");
		Document document2 = new Document("name", "wade");
		document2.append("age", 36);
		document2.append("sex", "男");
		Document document3 = new Document("name", "cp3");
		document3.append("age", 32);
		document3.append("sex", "男");
		
		List<Document> documents = new ArrayList<>();
		documents.add(document);
		documents.add(document2);
		documents.add(document3);
		
		collection.insertMany(documents);
		System.out.println("文档插入成功");  
	}
	@Test
	public void findDocuments() {
		MongoDatabase database = client.getDatabase("kkb");
		System.out.println("connect successful");
		MongoCollection<Document> collection = database.getCollection("mycollection2");
		System.out.println("获取集合成功："+collection.getNamespace());

		FindIterable<Document> iterable = collection.find();
		for (Document document : iterable) {
			System.out.println(document.toJson());
		}
		
//		MongoCursor<Document> mongoCursor = iterable.iterator();
//		while (mongoCursor.hasNext()) {
//			Document document = mongoCursor.next();
//			System.out.println(document.toJson());
//		}
	}
	@Test
	public void updateDocuments() {
		MongoDatabase database = client.getDatabase("kkb");
		System.out.println("connect successful");
		MongoCollection<Document> collection = database.getCollection("mycollection");
		System.out.println("获取集合成功："+collection.getNamespace());
		
		collection.updateMany(Filters.eq("age",18), new Document("$set", new Document("age", 20)));
		
		FindIterable<Document> iterable = collection.find();
		for (Document document : iterable) {
			System.out.println(document.toJson());
		}
		
	}
	@Test
	public void deleteDocuments() {
		MongoDatabase database = client.getDatabase("kkb");
		System.out.println("connect successful");
		MongoCollection<Document> collection = database.getCollection("mycollection");
		System.out.println("获取集合成功："+collection.getNamespace());
		
		// 删除符合条件的第一个文档 
		collection.deleteOne(Filters.eq("age",20));
		collection.deleteOne(Filters.eq("name","James"));
		// 删除符合条件的所有文档 
		collection.deleteMany(Filters.eq("age",20));
		 
		FindIterable<Document> iterable = collection.find();
		for (Document document : iterable) {
			System.out.println(document.toJson());
		}
		
	}
}

```

# spring-data-mongodb

## pom依赖

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.kkb</groupId>
	<artifactId>mongodb-demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<dependencies>
		<dependency>
			<groupId>org.mongodb</groupId>
			<artifactId>mongo-java-driver</artifactId>
			<version>3.10.1</version>
		</dependency>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-mongodb</artifactId>
			<version>2.1.1.RELEASE</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>5.0.7.RELEASE</version>
		</dependency>
    <dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<version>1.18.6</version>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.8.0</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>
```

## Spring配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:mongo="http://www.springframework.org/schema/data/mongo"
	xmlns:repository="http://www.springframework.org/schema/data/repository"
	xmlns:context="http://www.springframework.org/schema/context"

	xsi:schemaLocation="http://www.springframework.org/schema/beans
                    http://www.springframework.org/schema/beans/spring-beans.xsd
                    http://www.springframework.org/schema/context 
                    http://www.springframework.org/schema/context/spring-context.xsd
                    http://www.springframework.org/schema/data/mongo 
                    http://www.springframework.org/schema/data/mongo/spring-mongo.xsd
                    http://www.springframework.org/schema/data/repository 
                    http://www.springframework.org/schema/data/repository/spring-repository.xsd">

	<!-- 加载mongodb的配置属性文件 -->
	<context:property-placeholder location="classpath:mongodb.properties" />
	
	<!-- 扫描持久层 -->
	<context:component-scan base-package="com.kkb.mongodb.springdata" />
	
	<!-- 配置mongodb客户端连接服务器的相关信息 -->
	<mongo:mongo-client host="${mongo.host}" port="${mongo.port}" id="mongo">
		<mongo:client-options
			write-concern="${mongo.writeconcern}"
			connect-timeout="${mongo.connectTimeout}"
			socket-keep-alive="${mongo.socketKeepAlive}" />
	</mongo:mongo-client>

	<!-- mongo:db-factory dbname="database" mongo-ref="mongo" / -->
	<!--这里的dbname就是自己的数据库/collection的名字 -->
	<mongo:db-factory id="mongoDbFactory" dbname="kkb" mongo-ref="mongo" />

	<!-- 配置mongodb的模板类：MongoTemplate -->
	<bean id="mongoTemplate" class="org.springframework.data.mongodb.core.MongoTemplate">
		<constructor-arg name="mongoDbFactory" ref="mongoDbFactory" />
	</bean>

</beans>  
```

## properties

```properties
mongo.host=192.168.10.135
mongo.port=27017
mongo.connectionsPerHost=8
mongo.threadsAllowedToBlockForConnectionMultiplier=4
#连接超时时间
mongo.connectTimeout=1000
#等待时间
mongo.maxWaitTime=1500
mongo.autoConnectRetry=true
mongo.socketKeepAlive=true
#socket超时时间
mongo.socketTimeout=1500
mongo.slaveOK=true
mongo.writeconcern=safe
```

## po类

```java
package com.kkb.mongodb.springdata.po;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User {
	private String name;
	private int age;
	private String sex;
}

```

## dao接口和实现类

```java
package com.kkb.mongodb.springdata.dao;

import java.util.List;

import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;

import com.kkb.mongodb.springdata.po.User;
import com.mongodb.client.result.UpdateResult;

public interface UserDao {
	/**
	 * 根据条件查询
	 */
	public List<User> find(Query query);

	/**
	 * 根据条件查询一个
	 */
	public User findOne(Query query);

	/**
	 * 插入
	 */
	public void save(User entity);

	/**
	 * 根据条件 更新
	 */
	public UpdateResult update(Query query, Update update);

	/**
	 * 获得所有该类型记录,并且指定了集合名(表的意思)
	 */
	public List<User> findAll(String collectionName);

	/**
	 * 根据条件 获得总数
	 */
	public long count(Query query,String collectionName);

	/**
	 * 根据条件 删除
	 * 
	 * @param query
	 */
	public void remove(Query query);
}

```

```java
package com.kkb.mongodb.springdata.dao;

import java.util.List;

import javax.annotation.Resource;

import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;
import org.springframework.stereotype.Repository;

import com.kkb.mongodb.springdata.po.User;
import com.mongodb.client.result.UpdateResult;

@Repository
public class UserDaoImpl implements UserDao {

	@Resource
	private MongoTemplate template;
	
	@Override
	public List<User> find(Query query) {
		return template.find(query, User.class);
	}

	@Override
	public User findOne(Query query) {
		return template.findOne(query, User.class);
	}

	@Override
	public void save(User entity) {
		template.save(entity);
	}

	@Override
	public UpdateResult update(Query query, Update update) {
		return template.updateMulti(query, update, User.class);
	}

	@Override
	public List<User> findAll(String collectionName) {
		return template.findAll(User.class, collectionName);
	}

	@Override
	public long count(Query query,String collectionName) {
		return template.count(query, collectionName);
	}

	@Override
	public void remove(Query query) {
		template.remove(query,"user");
	}

}

```

## 测试类

```java
package com.kkb.mongodb.springdata.test;

import java.util.List;

import javax.annotation.Resource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import com.kkb.mongodb.springdata.dao.UserDao;
import com.kkb.mongodb.springdata.po.User;
import com.mongodb.client.result.UpdateResult;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:spring-mongodb.xml")
public class SpringDataMongodbTest {

	@Resource
	private UserDao userDao;

	@Test
	public void test1() {
		User user = new User();
		user.setName("科比");
		user.setAge(26);

		userDao.save(user);
		
		User user2 = new User();
		user2.setName("詹姆斯");
		user2.setAge(33);
		
		userDao.save(user2);
		
		User user3 = new User();
		user3.setName("韦德");
		user3.setAge(36);
		
		userDao.save(user3);
		
		User user4 = new User();
		user4.setName("罗斯");
		user4.setAge(30);
		
		userDao.save(user4);
		System.out.println("插入成功！");
	}

	@Test
	public void test2() {
		Query query = new Query(Criteria.where("name").is("科比"));
		List<User> list = userDao.find(query);
		for (User user : list) {
			System.out.println(user);
		}
		System.out.println("=======================");
		List<User> list2 = userDao.find(new Query());
		for (User user : list2) {
			System.out.println(user);
		}

		System.out.println("=======================");
		User user = (User) userDao.findOne(query);
		System.out.println(user);
	}

	@Test
	public void test3() {
		Query query = new Query(Criteria.where("name").is("韦德"));
		Update update = new Update();
		update.set("sex", "纯爷们");
		UpdateResult updateResult = userDao.update(query, update);
		System.out.println(updateResult.getUpsertedId());
	}

	@Test
	public void test4() {
		List<User> findAll = userDao.findAll("user");
		for (User user : findAll) {
			System.out.println(user);
		}
	}

	@Test
	public void test5() {
		Query query = new Query();
		long count = userDao.count(query,"user");
		System.out.println("总条数：" + count);
	}

	@Test
	public void test6() {
		Query query = new Query(Criteria.where("name").is("科比"));
		userDao.remove(query);
	}

}

```

