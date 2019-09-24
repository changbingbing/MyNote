### Mac OSX 平台安装 [MongoDB](http://www.runoob.com/mongodb/mongodb-tutorial.html)

1. 下载：[MongoDB 提供了 OSX 平台上 64 位的安装包](https://www.mongodb.com/download-center#community)

2. 接下来我们使用 curl 命令来下载安装：

   ```shell
   # 进入 /usr/local
   cd /usr/local
   
   # 下载
   sudo curl -O https://fastdl.mongodb.org/osx/mongodb-osx-x86_64-3.4.2.tgz
   
   # 解压
   sudo tar -zxvf mongodb-osx-x86_64-3.4.2.tgz
   
   # 重命名为 mongodb 目录
   
   sudo mv mongodb-osx-x86_64-3.4.2 mongodb
   ```

   安装完成后，我们可以把 MongoDB 的二进制命令文件目录（安装目录/bin）添加到 PATH 路径中：

   ```shell
   #编辑
   vi /etc/profile
   
   #添加到 PATH 路径
   export PATH=/usr/local/mongodb/bin:$PATH
   
   #保存，退出，使之生效:
   source /etc/profile
   ```

### 运行MongoDB

1. 首先我们创建一个数据库存储目录 /data/db：

   ```shell
   sudo mkdir -p /data/db
   ```

2. 启动 mongodb，默认数据库目录即为 /data/db：

   ```shell
   sudo mongod
   
   # 如果没有创建全局路径 PATH，需要进入以下目录
   cd /usr/local/mongodb/bin
   sudo ./mongod
   ```

3. 再打开一个终端进入执行以下命令：

   ```shell
   $ cd /usr/local/mongodb/bin 
   $ ./mongo
   MongoDB shell version v3.4.2
   connecting to: mongodb://127.0.0.1:27017
   MongoDB server version: 3.4.2
   Welcome to the MongoDB shell.
   ……
   > 1 + 1
   2
   > 
   ```

⚠️：

> *注意：如果你的数据库目录不是/data/db，可以通过 --dbpath 来指定。*

