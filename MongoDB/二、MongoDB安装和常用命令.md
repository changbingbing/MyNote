# [安装](https://www.runoob.com/mongodb/mongodb-linux-install.html)

MongoDB 提供了 linux 各发行版本 64 位的安装包，你可以在官网下载安装包。

下载地址：https://www.mongodb.com/download-center#community

![](http://pwtosjisl.bkt.clouddn.com/ipic-blog/2019-09-05-070546.png)

下载完安装包，并解压 **tgz**（以下演示的是 64 位 Linux上的安装） 。

```shell
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-4.2.0.tgz
tar -zxvf mongodb-linux-x86_64-rhel70-4.2.0.tgz -C /usr/apps/
mv mongodb-linux-x86_64-rhel70-4.2.0 mongodb

# 一定注意要创建数据目录,默认路径/data/db，这个目录在安装过程不会自动创建，所以你需要手动创建data目录，并在data目录中创建db目录。注意：/data/db 是 MongoDB 默认的启动的数据库路径(--dbpath)。
mkdir -p /data/db
```

MongoDB 的可执行文件位于 bin 目录下，所以可以将其添加到 **PATH** 路径中：

```shell
#<mongodb-install-directory> 为你 MongoDB 的安装路径。如本文的 /usr/apps/mongodb 。
export PATH=<mongodb-install-directory>/bin:$PATH
```

# 启动

* 通过配置文件方式启动

  ```shell
  #-f是--config的缩写，配置文件需要手动创建，参考以下示例即可
  mongod --config <配置文件路径>
  mongod -f <配置文件路径>
  ```

* 创建数据和日志目录:

  ```shell
  mkdir /root/mongodb/singleton/data -p
  mkdir /root/mongodb/singleton/logs -p
  ```

* 创建mongodb.cfg配置文件

  ```shell
  #数据库文件位置 
  dbpath=/root/mongodb/singleton/data
  #日志文件位置 
  logpath=/root/mongodb/singleton/logs/mongodb.log
  # 以追加方式写入日志
  logappend=true
  # 是否以守护进程方式运行
  fork=true
  #绑定客户端访问的ip
  bind_ip=192.168.36.230
  # 默认27017
  port=27017
  ```

* 通过配置文件方式(后台进程方式启动)启动:

  ```shell
  mongod -f /root/mongodb/sinleton/mongodb.cfg
  ```

# 连接客户端

```shell
mongo 192.168.36.230:27017
```

# 常用客户端

* 自带命令行客户端
* Navicat for mongodb
* MongoVUE
* Studio 3T
* **Robo 3T**
* RockMongo

# 常用命令

## [创建数据库](https://www.runoob.com/mongodb/mongodb-create-database.html)

### 语法

MongoDB 创建数据库的语法格式如下：

```sql
use DATABASE_NAME
```

如果数据库不存在，则创建数据库，否则切换到指定数据库。

### 实例

以下实例我们创建了数据库 runoob:

```sql
> use runoob
switched to db runoob
> db
runoob
> 
```

如果你想查看所有数据库，可以使用 **show dbs** 命令：

```sql
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
> 
```

可以看到，我们刚创建的数据库 runoob 并不在数据库的列表中， 要显示它，我们需要向 runoob 数据库插入一些数据。

```sql
> db.runoob.insert({"name":"Robin"})
WriteResult({ "nInserted" : 1 })
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
runoob  0.000GB
```

MongoDB 中默认的数据库为 test，如果你没有创建新的数据库，集合将存放在 test 数据库中。

> **注意:** *在 MongoDB 中，集合只有在内容插入后才会创建! 就是说，创建集合(数据表)后要再插入一个文档(记录)，集合才会真正创建。*

## [删除数据库](https://www.runoob.com/mongodb/mongodb-dropdatabase.html)

### 语法

MongoDB 删除数据库的语法格式如下：

```sql
db.dropDatabase()
```

删除当前数据库，默认为 test，你可以使用 db 命令查看当前数据库名。

### 实例

以下实例我们删除了数据库 runoob。

首先，查看所有数据库：

```sql
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
runoob  0.000GB
```

接下来我们切换到数据库 runoob：

```sql
> use runoob
switched to db runoob
>
```

执行删除命令：

```sql
> db.dropDatabase()
{ "dropped" : "runoob", "ok" : 1 }
```

最后，我们再通过 show dbs 命令数据库是否删除成功：

```sql
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```

## [创建集合](https://www.runoob.com/mongodb/mongodb-create-collection.html)

MongoDB 中使用 **createCollection()** 方法来创建集合。

语法格式：

```s q
db.createCollection(name, options)
```

参数说明：

* name: 要创建的集合名称
* options: 可选参数, 指定有关内存大小及索引的选项

options 可以是如下参数：

| 字段        | 类型 | 描述                                                         |
| :---------- | :--- | :----------------------------------------------------------- |
| capped      | 布尔 | （可选）如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。 **当该值为 true 时，必须指定 size 参数。** |
| autoIndexId | 布尔 | （可选）如为 true，自动在 _id 字段创建索引。默认为 false。   |
| size        | 数值 | （可选）为固定集合指定一个最大值（以字节计）。 **如果 capped 为 true，也需要指定该字段。** |
| max         | 数值 | （可选）指定固定集合中包含文档的最大数量。                   |

在插入文档时，MongoDB 首先检查固定集合的 size 字段，然后检查 max 字段。

### 实例

在 test 数据库中创建 runoob 集合：

```sql
> use test
switched to db test
> db.createCollection("runoob")
{ "ok" : 1 }
>
```

如果要查看已有集合，可以使用 **show collections** 或 **show tables** 命令：

```
> show collections
runoob
system.indexes
```

下面是带有几个关键参数的 createCollection() 的用法：

创建固定集合 mycol，整个集合空间大小 6142800 KB, 文档最大个数为 10000 个。

```sql
> db.createCollection("mycol", { capped : true, autoIndexId : true, size : 
   6142800, max : 10000 } )
{ "ok" : 1 }
>
```

在 MongoDB 中，你不需要创建集合。当你插入一些文档时，MongoDB 会自动创建集合。

```sql
> db.mycol2.insert({"name" : "Robin2"})
> show collections
mycol2
...
```

## [删除集合](https://www.runoob.com/mongodb/mongodb-delete-collection.html)

MongoDB 中使用 drop() 方法来删除集合。

**语法格式：**

```sql
db.collection.drop()
```

**返回值**

如果成功删除选定集合，则 drop() 方法返回 true，否则返回 false。

以下实例删除了 runoob 数据库中的集合 Robin:

```sql
> use runoob
switched to db runoob
> show tables
Robin
> db.Robin.drop()
true
> show tables
>
```

## [插入文档](https://www.runoob.com/mongodb/mongodb-insert.html)

> 文档的数据结构和 JSON 基本一样。
>
> 所有存储在集合中的数据都是 BSON 格式。
>
> BSON 是一种类似 JSON 的二进制形式的存储格式，是 Binary JSON 的简称。

MongoDB 使用 insert() 或 save() 方法向集合中插入文档,语法如下:

```sql
db.COLLECTION_NAME.insert(document)
-- 插入文档你也可以使用 db.col.save(document) 命令。如果不指定 _id 字段 save() 方法类似于 insert() 方法。如果指定 _id 字段，则会更新该 _id 的数据。
```

以下文档可以存储在 MongoDB 的 mymongo 数据库 的 mycollection 集合中:

```sql
>db.mycollection.insert({title: 'MongoDB 教程', 
    description: 'MongoDB 是一个 Nosql 数据库',
    by: '菜鸟教程',
    url: 'http://www.runoob.com',
    tags: ['mongodb', 'database', 'NoSQL'],
    likes: 100
})
-- 以上实例中 mycollection 是我们的集合名，如果该集合不在该数据库中， MongoDB 会自动创建该集合并插入文档。


> document=({title: 'MongoDB 教程', 
    description: 'MongoDB 是一个 Nosql 数据库',
    by: '菜鸟教程',
    url: 'http://www.runoob.com',
    tags: ['mongodb', 'database', 'NoSQL'],
    likes: 100
});
> db.mycollection.insert(document)
WriteResult({ "nInserted" : 1 })


-- 查看已插入文档
> db.col.find()

```

## [更新文档](https://www.runoob.com/mongodb/mongodb-update.html)

​	MongoDB 使用 **update()** 和 **save()** 方法来更新集合中的文档。接下来让我们详细来看下两个函数的应用及其区别。

### update()方法

update() 方法用于更新已存在的文档。语法格式如下：

```sql
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)
```

**参数说明：**

* **query** : update的查询条件，类似sql update查询内where后面的。
* **update** : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
* **upsert** : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
* **multi** : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
* **writeConcern** :可选，抛出异常的级别。

#### 实例

在集合mycollection中插入数据：

```sql
>db.mycollection.insert({
    title: 'MongoDB 教程', 
    description: 'MongoDB 是一个 Nosql 数据库',
    by: '菜鸟教程',
    url: 'http://www.runoob.com',
    tags: ['mongodb', 'database', 'NoSQL'],
    likes: 100
})
```

通过 update() 方法来更新标题(title):

```sql
>db.mycollection.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })   --输出信息
```

可以看到标题(title)由原来的 "MongoDB 教程" 更新为了 "MongoDB"。

以上语句只会修改第一条发现的文档，如果你要修改多条相同的文档，则需要设置 multi 参数为 true。

```sql
>db.mycollection.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}},{multi:true})
```

#### 更多实例

只更新第一条记录：

```
db.col.update( { "count" : { $gt : 1 } } , { $set : { "test2" : "OK"} } );
```

全部更新：

```sql
db.col.update( { "count" : { $gt : 3 } } , { $set : { "test2" : "OK"} },false,true );
```

只添加第一条：

```sql
db.col.update( { "count" : { $gt : 4 } } , { $set : { "test5" : "OK"} },true,false );
```

全部添加进去:

```sql
db.col.update( { "count" : { $gt : 5 } } , { $set : { "test5" : "OK"} },true,true );
```

全部更新：

```sql
db.col.update( { "count" : { $gt : 15 } } , { $inc : { "count" : 1} },false,true );
```

只更新第一条记录：

```sql
db.col.update( { "count" : { $gt : 10 } } , { $inc : { "count" : 1} },false,false );
```

### save()方法

save() 方法通过传入的文档来替换已有文档。语法格式如下：

```sql
db.collection.save(
   <document>,
   {
     writeConcern: <document>
   }
)
```

**参数说明：**

* **document** : 文档数据。
* **writeConcern** :可选，抛出异常的级别。

#### 实例

替换了 _id 为 56064f89ade2f21f36b03136 的文档数据：

```sql
>db.mycollection.save({
    "_id" : ObjectId("56064f89ade2f21f36b03136"),
    "title" : "MongoDB",
    "description" : "MongoDB 是一个 Nosql 数据库",
    "by" : "Runoob",
    "url" : "http://www.runoob.com",
    "tags" : [
            "mongodb",
            "NoSQL"
    ],
    "likes" : 110
})

>db.col.find().pretty()
```

## [删除文档](https://www.runoob.com/mongodb/mongodb-remove.html)

​	MongoDB remove()函数是用来移除集合中的数据。在执行remove()函数前先执行find()命令来判断执行的条件是否正确，这是一个比较好的习惯。

### 语法

remove() 方法的基本语法格式如下所示：

```sql
db.collection.remove(
   <query>,
   <justOne>
)
```

如果你的 MongoDB 是 2.6 版本以后的，语法格式如下：

```sql
db.collection.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
```

**参数说明：**

* **query** :（可选）删除的文档的条件。
* **justOne** : （可选）如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配条件的文档。
* **writeConcern** :（可选）抛出异常的级别。

### 实例

以下文档我们执行两次插入操作：

```sql
>db.col.insert({title: 'MongoDB 教程', 
    description: 'MongoDB 是一个 Nosql 数据库',
    by: '菜鸟教程',
    url: 'http://www.runoob.com',
    tags: ['mongodb', 'database', 'NoSQL'],
    likes: 100
})
```

使用 find() 函数查询数据

```sql
> db.col.find()
{ "_id" : ObjectId("56066169ade2f21f36b03137"), "title" : "MongoDB 教程", "description" : "MongoDB 是一个 Nosql 数据库", "by" : "菜鸟教程", "url" : "http://www.runoob.com", "tags" : [ "mongodb", "database", "NoSQL" ], "likes" : 100 }
{ "_id" : ObjectId("5606616dade2f21f36b03138"), "title" : "MongoDB 教程", "description" : "MongoDB 是一个 Nosql 数据库", "by" : "菜鸟教程", "url" : "http://www.runoob.com", "tags" : [ "mongodb", "database", "NoSQL" ], "likes" : 100 }
```

接下来我们移除 title 为 'MongoDB 教程' 的文档：

```sql
>db.col.remove({'title':'MongoDB 教程'})
WriteResult({ "nRemoved" : 2 })           -- 删除了两条数据
>db.col.find()
……                                        -- 没有数据

```

如果你只想删除第一条找到的记录可以设置 justOne 为 1，如下所示：

```sql
>db.COLLECTION_NAME.remove(DELETION_CRITERIA,1)
```

如果你想删除所有数据，可以使用以下方式（类似常规 SQL 的 truncate 命令）：

```sql
>db.col.remove({})
>db.col.find()
>
```

### 补充

* remove() 方法已经过时了，现在官方推荐使用 deleteOne() 和 deleteMany() 方法。

  * 如删除集合下全部文档：

    ```sql
    db.inventory.deleteMany({})
    ```

  * 删除 status 等于 A 的全部文档：

    ```sql
    db.inventory.deleteMany({ status : "A" })
    ```

  * 删除 status 等于 D 的一个文档：

    ```sql
    db.inventory.deleteOne( { status: "D" } )
    ```

* remove() 方法 并不会真正释放空间。需要继续执行 db.repairDatabase() 来回收磁盘空间。

  ```sql
  > db.repairDatabase()
  或者
  > db.runCommand({ repairDatabase: 1 })
  ```

## [查询文档](https://www.runoob.com/mongodb/mongodb-query.html)

### 语法

MongoDB 查询数据的语法格式如下：

```sql
db.collection.find(query, projection)
```

* **query** ：可选，使用查询操作符指定查询条件
* **projection** ：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略）。

如果你需要以易读的方式来读取数据，可以使用 pretty() 方法，以格式化的方式来显示所有文档，语法格式如下：

```sql
>db.col.find().pretty()
```

### 实例

以下实例我们查询了集合 col 中的数据：

```sql
> db.col.find().pretty()
{
        "_id" : ObjectId("56063f17ade2f21f36b03133"),
        "title" : "MongoDB 教程",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "菜鸟教程",
        "url" : "http://www.runoob.com",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "likes" : 100
}
```

除了 find() 方法之外，还有一个 findOne() 方法，它只返回一个文档。

### MongoDB与RDBMS Where语句比较

如果你熟悉常规的 SQL 数据，通过下表可以更好的理解 MongoDB 的条件语句查询：

| 操作       | 格式                     | 范例                                        | RDBMS中的类似语句       |
| :--------- | :----------------------- | :------------------------------------------ | :---------------------- |
| 等于       | `{<key>:<value>`}        | `db.col.find({"by":"菜鸟教程"}).pretty()`   | `where by = '菜鸟教程'` |
| 小于       | `{<key>:{$lt:<value>}}`  | `db.col.find({"likes":{$lt:50}}).pretty()`  | `where likes < 50`      |
| 小于或等于 | `{<key>:{$lte:<value>}}` | `db.col.find({"likes":{$lte:50}}).pretty()` | `where likes <= 50`     |
| 大于       | `{<key>:{$gt:<value>}}`  | `db.col.find({"likes":{$gt:50}}).pretty()`  | `where likes > 50`      |
| 大于或等于 | `{<key>:{$gte:<value>}}` | `db.col.find({"likes":{$gte:50}}).pretty()` | `where likes >= 50`     |
| 不等于     | `{<key>:{$ne:<value>}}`  | `db.col.find({"likes":{$ne:50}}).pretty()`  | `where likes != 50`     |

### MongoDB AND条件

MongoDB 的 find() 方法可以传入多个键(key)，每个键(key)以逗号隔开，即常规 SQL 的 AND 条件。

语法格式如下：

```sql
>db.col.find({key1:value1, key2:value2}).pretty()
```

以下实例通过 **by** 和 **title** 键来查询 **菜鸟教程** 中 **MongoDB 教程** 的数据

```sql
> db.col.find({"by":"菜鸟教程", "title":"MongoDB 教程"}).pretty()
{
        "_id" : ObjectId("56063f17ade2f21f36b03133"),
        "title" : "MongoDB 教程",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "菜鸟教程",
        "url" : "http://www.runoob.com",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "likes" : 100
}
```

以上实例中类似于 WHERE 语句：**WHERE by='菜鸟教程' AND title='MongoDB 教程'**

### MongoDB AND条件

MongoDB OR 条件语句使用了关键字 **$or**,语法格式如下：

```sql
>db.col.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()
```

以下实例中，我们演示了查询键 **by** 值为 菜鸟教程 或键 **title** 值为 **MongoDB 教程** 的文档。

```sql
>db.col.find({$or:[{"by":"菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()
{
        "_id" : ObjectId("56063f17ade2f21f36b03133"),
        "title" : "MongoDB 教程",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "菜鸟教程",
        "url" : "http://www.runoob.com",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "likes" : 100
}

```

### AND和OR联合使用

以下实例演示了 AND 和 OR 联合使用，类似常规 SQL 语句为： **'where likes>50 AND (by = '菜鸟教程' OR title = 'MongoDB 教程')'**

```sql
>db.col.find({"likes": {$gt:50}, $or: [{"by": "菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()
{
        "_id" : ObjectId("56063f17ade2f21f36b03133"),
        "title" : "MongoDB 教程",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "菜鸟教程",
        "url" : "http://www.runoob.com",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "likes" : 100
}
```

## [条件操作符](https://www.runoob.com/mongodb/mongodb-operators.html)

## [$type操作符](https://www.runoob.com/mongodb/mongodb-operators-type.html)

## [limit与skip方法](https://www.runoob.com/mongodb/mongodb-limit-skip.html)

### Limit() 方法

如果你需要在MongoDB中读取指定数量的数据记录，可以使用MongoDB的Limit方法，limit()方法接受一个数字参数，该参数指定从MongoDB中读取的记录条数。

#### 语法

limit()方法基本语法如下所示：

```sql
>db.COLLECTION_NAME.find().limit(NUMBER)
```

#### 实例

集合col中的数据如下：

```sql
{ "_id" : ObjectId("56066542ade2f21f36b0313a"), "title" : "PHP 教程", "description" : "PHP 是一种创建动态交互性站点的强有力的服务器端脚本语言。", "by" : "菜鸟教程", "url" : "http://www.runoob.com", "tags" : [ "php" ], "likes" : 200 }
{ "_id" : ObjectId("56066549ade2f21f36b0313b"), "title" : "Java 教程", "description" : "Java 是由Sun Microsystems公司于1995年5月推出的高级程序设计语言。", "by" : "菜鸟教程", "url" : "http://www.runoob.com", "tags" : [ "java" ], "likes" : 150 }
{ "_id" : ObjectId("5606654fade2f21f36b0313c"), "title" : "MongoDB 教程", "description" : "MongoDB 是一个 Nosql 数据库", "by" : "菜鸟教程", "url" : "http://www.runoob.com", "tags" : [ "mongodb" ], "likes" : 100 }
```

以下实例为显示查询文档中的两条记录：

```sql
> db.col.find({},{"title":1,_id:0}).limit(2)
{ "title" : "PHP 教程" }
{ "title" : "Java 教程" }
>
```

注：如果你们没有指定limit()方法中的参数则显示集合中的所有数据。

### Skip() 方法

使用skip()方法来跳过指定数量的数据，skip方法同样接受一个数字参数作为跳过的记录条数。

#### 语法

skip() 方法脚本语法格式如下：

```sql
>db.COLLECTION_NAME.find().limit(NUMBER).skip(NUMBER)
```

#### 实例

以下实例只会显示第二条文档数据

```sql
>db.col.find({},{"title":1,_id:0}).limit(1).skip(1)
{ "title" : "Java 教程" }
>
```

**注:**skip()方法默认参数为 0 。

## [文档排序](https://www.runoob.com/mongodb/mongodb-sort.html)

​	在 MongoDB 中使用 sort() 方法对数据进行排序，sort() 方法可以通过参数指定排序的字段，并使用 1 和 -1 来指定排序的方式，其中 1 为升序排列，而 -1 是用于降序排列。

### 语法

```sql
>db.COLLECTION_NAME.find().sort({KEY:1})
```

### 实例

col 集合中的数据如下：

```sql
{ "_id" : ObjectId("56066542ade2f21f36b0313a"), "title" : "PHP 教程", "description" : "PHP 是一种创建动态交互性站点的强有力的服务器端脚本语言。", "by" : "菜鸟教程", "url" : "http://www.runoob.com", "tags" : [ "php" ], "likes" : 200 }
{ "_id" : ObjectId("56066549ade2f21f36b0313b"), "title" : "Java 教程", "description" : "Java 是由Sun Microsystems公司于1995年5月推出的高级程序设计语言。", "by" : "菜鸟教程", "url" : "http://www.runoob.com", "tags" : [ "java" ], "likes" : 150 }
{ "_id" : ObjectId("5606654fade2f21f36b0313c"), "title" : "MongoDB 教程", "description" : "MongoDB 是一个 Nosql 数据库", "by" : "菜鸟教程", "url" : "http://www.runoob.com", "tags" : [ "mongodb" ], "likes" : 100 }
```

以下实例演示了 col 集合中的数据按字段 likes 的降序排列：

```sql
>db.col.find({},{"title":1,_id:0}).sort({"likes":-1})
{ "title" : "PHP 教程" }
{ "title" : "Java 教程" }
{ "title" : "MongoDB 教程" }
>
```

## [MongoDB索引](https://www.runoob.com/mongodb/mongodb-indexing.html)

​	索引通常能够极大的提高查询的效率，如果没有索引，MongoDB在读取数据时必须扫描集合中的每个文件并选取那些符合查询条件的记录。

​	这种扫描全集合的查询效率是非常低的，特别在处理大量的数据时，查询可以要花费几十秒甚至几分钟，这对网站的性能是非常致命的。

​	索引是特殊的数据结构，索引存储在一个易于遍历读取的数据集合中，索引是对数据库表中一列或多列的值进行排序的一种结构。

### 创建索引

MongoDB使用 createIndex() 方法来创建索引。

> *注意在 3.0.0 版本前创建索引方法为 db.collection.ensureIndex()，之后的版本使用了db.collection.createIndex() 方法，ensureIndex() 还能用，但只是 createIndex() 的别名。*

#### 语法

createIndex()方法基本语法格式如下所示：

```sql
>db.collection.createIndex(keys, options)
```

语法中 Key 值为你要创建的索引字段，1 为指定按升序创建索引，如果你想按降序来创建索引指定为 -1 即可。

#### 实例1

```sql
>db.col.createIndex({"title":1})
>
```

createIndex() 方法中你也可以设置使用多个字段创建索引（关系型数据库中称作复合索引）。

```sql
>db.col.createIndex({"title":1,"description":-1})
>
```

createIndex() 接收可选参数，可选参数列表如下：

| Parameter          | Type          | Description                                                  |
| :----------------- | :------------ | :----------------------------------------------------------- |
| background         | Boolean       | 建索引过程会阻塞其它数据库操作，background可指定以后台方式创建索引，即增加 "background" 可选参数。 "background" 默认值为**false**。 |
| unique             | Boolean       | 建立的索引是否唯一。指定为true创建唯一索引。默认值为**false**. |
| name               | string        | 索引的名称。如果未指定，MongoDB的通过连接索引的字段名和排序顺序生成一个索引名称。 |
| dropDups           | Boolean       | **3.0+版本已废弃。**在建立唯一索引时是否删除重复记录,指定 true 创建唯一索引。默认值为 **false**. |
| sparse             | Boolean       | 对文档中不存在的字段数据不启用索引；这个参数需要特别注意，如果设置为true的话，在索引字段中不会查询出不包含对应字段的文档.。默认值为 **false**. |
| expireAfterSeconds | integer       | 指定一个以秒为单位的数值，完成 TTL设定，设定集合的生存时间。 |
| v                  | index version | 索引的版本号。默认的索引版本取决于mongod创建索引时运行的版本。 |
| weights            | document      | 索引权重值，数值在 1 到 99,999 之间，表示该索引相对于其他索引字段的得分权重。 |
| default_language   | string        | 对于文本索引，该参数决定了停用词及词干和词器的规则的列表。 默认为英语 |
| language_override  | string        | 对于文本索引，该参数指定了包含在文档中的字段名，语言覆盖默认的language，默认值为 language. |

#### 实例2

在后台创建索引：

```sql
db.values.createIndex({open: 1, close: 1}, {background: true})
```

通过在创建索引时加 `background:true` 的选项，让创建工作在后台执行。

### 查看集合索引

```sql
db.col.getIndexes()
```

### 查看集合索引大小

```sql
db.col.totalIndexSize()
```

### 删除集合所有索引

```sql
db.col.dropIndexes()
```

### 删除集合指定索引

```sql
db.col.dropIndex("索引名称")
```

## [聚合查询](https://www.runoob.com/mongodb/mongodb-aggregate.html)

参考文章：https://www.cnblogs.com/zhoujie/p/mongo1.html

​	MongoDB中聚合(aggregate)主要用于处理数据(诸如统计平均值,求和等)，并返回计算后的数据结果。有点类似sql语句中的 count(*)。

### 语法

aggregate() 方法的基本语法格式如下所示：

```sql
>db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)
```

### 实例

集合中插入数据如下：

```sql
> db.mycollection.insert({title:'MongoDB Overview',description:'MongoDB is no sql database',by_user:'runoob.com',url:'http://www.runoob.com',tags:['mongodb','database','NoSQL'],likes:100})
> db.mycollection.insert({title:'NoSQL Overview',description:'No sql database is very fast',by_user:'runoob.com',url:'http://www.runoob.com',tags:['mongodb','database','NoSQL'],likes:10})
> db.mycollection.insert({title:'Neo4j Overview',description:'Neo4j is no sql database',by_user:'Neo4j',url:'http://www.neo4j.com',tags:['neo4j','database','NoSQL'],likes:750})
```

现在我们通过以上集合计算每个作者所写的文章数，使用aggregate()计算结果如下：

```sql
> db.mycollection.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : 1}}}])
{ "_id" : "runoob.com", "num_tutorial" : 2 }
{ "_id" : "Neo4j", "num_tutorial" : 1 }
>
```

以上实例类似sql语句：

```sql
select by_user, count(*) from mycol group by by_user
```

在上面的例子中，我们通过字段 by_user 字段对数据进行分组，并计算 by_user 字段相同值的总和。

下表展示了一些聚合的表达式:

| 表达式    | 描述                                           | 实例                                                         |
| :-------- | :--------------------------------------------- | :----------------------------------------------------------- |
| $sum      | 计算总和。                                     | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : "$likes"}}}]) |
| $avg      | 计算平均值                                     | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$avg : "$likes"}}}]) |
| $min      | 获取集合中所有文档对应值得最小值。             | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$min : "$likes"}}}]) |
| $max      | 获取集合中所有文档对应值得最大值。             | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$max : "$likes"}}}]) |
| $push     | 在结果文档中插入值到一个数组中。               | db.mycol.aggregate([{$group : {_id : "$by_user", url : {$push: "$url"}}}]) |
| $addToSet | 在结果文档中插入值到一个数组中，但不创建副本。 | db.mycol.aggregate([{$group : {_id : "$by_user", url : {$addToSet : "$url"}}}]) |
| $first    | 根据资源文档的排序获取第一个文档数据。         | db.mycol.aggregate([{$group : {_id : "$by_user", first_url : {$first : "$url"}}}]) |
| $last     | 根据资源文档的排序获取最后一个文档数据         | db.mycol.aggregate([{$group : {_id : "$by_user", last_url : {$last : "$url"}}}]) |

## [MongoDB 复制（副本集）](https://www.runoob.com/mongodb/mongodb-replication.html)

## [MongoDB 分片](https://www.runoob.com/mongodb/mongodb-sharding.html)

