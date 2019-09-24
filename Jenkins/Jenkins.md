jenkins是devops神器，这里对如何安装和使用jenkins部署应用（war）做一个整理(Mac环境)。
jenkins搭建部署主要分为4个步骤：
* 第一步，jenkins安装
* 第二步，插件安装和配置
* 第三步，Push SSH
* 第四步，部署项目
# 第一步，jenkins安装
## 准备环境：
* JDK：1.8
* Jenkins：2.153
* Mac：10.11
* maven：3.39
> JDK、maven默认已安装完成
## 配置防护墙
Mac安装未配防护墙
```shell
#centos7
systemctl stop firewalld.service
==============================
#以下为：centOS 6.5关闭防火墙步骤
#关闭命令：  
service iptables stop 
#永久关闭防火墙：
chkconfig iptables off
```
两个命令同时运行，运行完成后查看防火墙关闭状态
```shell
service iptables status
```
## jenkins安装
* 下载[Jenkins](https://jenkins.io/download/),双击jenkins-2.121.2.pkg进行安装或
```shell
#brew search jenkins
brew install jenkins
```
* 启动服务
```shell
brew services start jenkins
#或
jenkins
```
Jenkins 就启动成功了！它的war包自带Jetty服务器

第一次启动Jenkins时，出于安全考虑，Jenkins会自动生成一个随机的按照口令。**注意控制台输出的口令，复制下来**，然后在浏览器输入密码：
```shell
INFO: 

*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

0cca37389e6540c08ce6e4c96f46da0f

This may also be found at: /root/.jenkins/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************
```
* 浏览器访问：[http://localhost:8080/](http://localhost:8080/)

![1.png](https://ws2.sinaimg.cn/large/006tNbRwgy1fxx263ijv7j30rk0k5aaw.jpg)
输入：0cca37389e6540c08ce6e4c96f46da0f

进入用户自定义插件界面，建议选择安装官方推荐插件，因为安装后自己也得安装:
![2.png](https://ws4.sinaimg.cn/large/006tNbRwgy1fxx2729oxqj30k50dwaea.jpg)
* 接下来是进入插件安装进度界面:
![3.png](https://ws4.sinaimg.cn/large/006tNbRwgy1fxx27dy9cuj30rm0kcwfp.jpg)
插件一次可能不会完全安装成功，可以点击Retry再次安装。直到全部安装成功
![32.png](https://ws4.sinaimg.cn/large/006tNbRwgy1fxx27p1ymuj30rp0kc40e.jpg)
* 等待一段时间之后，插件安装完成，配置用户名密码:
![4.png](https://ws1.sinaimg.cn/large/006tNbRwgy1fxx27xxslpj30rk0kcjrr.jpg)
输入：admin/admin
# 第二步，插件安装和配置
> 有很多插件都是选择的默认的安装的，所以现在需要我们安装的插件不多，Git plugin和Maven Integration plugin，publish over SSH。

插件安装：系统管理 > 插件管理 > 可选插件,勾选需要安装的插件，点击直接安装或者下载重启后安装
![7.png](https://ws2.sinaimg.cn/large/006tNbRwgy1fxx28qvvdrj312k0lf0u2.jpg)
## 配置全局变量
* 系统管理-》全局工具配置(Maven Configuration、JDK、Maven...)
![5.png](https://ws1.sinaimg.cn/large/006tNbRwgy1fxx2854izhj31ae0u078z.jpg)
![6.png](https://ws3.sinaimg.cn/large/006tNbRwgy1fxx28is7j6j31ea0e4abh.jpg)
其它内容可以根据自己的情况选择安装
## 配置SSH面登陆
ssh的配置可使用密钥，也可以使用密码，这里我们使用密钥来配置，在配置之前先配置好jenkins服务器和应用服务器的密钥认证 
* **jenkins服务器**上生成密钥对，使用`ssh-keygen -t rsa`命令
  
    输入下面命令 一直回车，一个矩形图形出现就说明成功，在~/.ssh/下会有私钥id_rsa和公钥id_rsa.pub
    
```shell
ssh-keygen -t rsa
```
* 将**jenkins**服务器的公钥`id_rsa.pub`中的内容复制到**应用服务器**的~/.ssh/下的`authorized_keys`文件
```shell
#ssh-copy-id - 将你的公共密钥填充到一个远程机器上的authorized_keys文件中。
ssh-copy-id -i id_rsa.pub 192.168.0.xx
chmod 644 authorized_keys
```
* 在**应用服务器上**重启ssh服务，`service sshd restart`现在jenkins服务器可免密码直接登陆应用服务器

    之后在用ssh B尝试能否免密登录B服务器，如果还是提示需要输入密码，则有以下原因:
    * a. 非root账户可能不支持ssh公钥认证（看服务器是否有限制）
    * b. 传过来的公钥文件权限不够，可以给这个文件授权下 `chmod 644 authorized_keys`
    * c. 使用root账户执行`ssh-copy-id -i ~/.ssh/id_rsa.pub` 这个指令的时候如果需要输入密码则要配置sshd_config
```shell
vi /etc/ssh/sshd_config
#内容
PermitRootLogin no
```
修改之后需要重启sshd服务
```shell
service sshd restart
```
* 最后，如果可以SSH IP 免密登录成功说明SSH公钥认证成功。
# 第三步，Push SSH
* 系统管理 > 系统设置（配置Publish over SSH）
![Publish over SSH](https://ws2.sinaimg.cn/large/006tNbRwgy1fxx3mog065j30p70jft9u.jpg)
    * Passphrase 秘钥的密码，设置了则填写，不用设置
    * Path to key 客户端私钥路径，写上生成的ssh路径：`/Users/***/.ssh/id_rsa`
    * **Name 随意起名代表这个服务，待会要根据它来选择**
    * **Hostname 配置应用服务器的IP地址** 
    * **Username 配置应用服务器linux登陆用户名－远程机器用户名**
    * **Remote Directory 可填可不填 ⚠️：此处(系统设置SSH Server)的Remote Directory＋部署中SSH Publishers配置的Remote directory，即为应用服务器全路径**
> 点击下方`Add`可以添加多个应用服务器的地址
# 第四步，部署项目
* 首先，**新建**输入项目名称，选择构建一个maven项目，点击确定。

    ![新建](https://ws2.sinaimg.cn/large/006tNbRwgy1fxx480xs35j311d0kd769.jpg)
* 勾选**丢弃旧的构建**，选择是否备份被替换的旧包。我这里选择备份最近的10个

    ![丢弃旧的构建](https://ws4.sinaimg.cn/large/006tNbRwgy1fxx4ae4r9aj30vo0j7wf6.jpg)
* 源码管理,选择Git,配置Git相关信息，点击add可以输入git的账户和密码

    ![配置Git](https://ws2.sinaimg.cn/large/006tNbRwgy1fxx4dlzqx0j31ee0u0q6o.jpg)
* 构建环境中勾选**Add timestamps to the Console Output**，代码构建的过程中会将日志打印出来

    ![构建日志输出配置](https://ws1.sinaimg.cn/large/006tNbRwgy1fxx4gmfouyj30sf07xaa9.jpg)
* 在Build中输入打包前的mvn命令，如：
`clean install -Dmaven.test.skip=true -Ptest`
  
    ![mvn](https://ws3.sinaimg.cn/large/006tNbRwgy1fxx4lvuaz5j31fa09gt9q.jpg)
* Post Steps 选择 **Run only if build succeeds**

    ![Post Steps](https://ws4.sinaimg.cn/large/006tNbRwgy1fxx4nwjl28j30r004uglj.jpg)
* 点击Add **post-build step**，选择 `Send files or execute commands over SSH`

    ![post-build step](https://ws3.sinaimg.cn/large/006tNbRwgy1fxx4qojnxgj31ey0dcdhi.jpg)
* Name选择上面配置的Push SSH

    ![Push SSH](https://ws1.sinaimg.cn/large/006tNbRwgy1fxx4vlfd98j31dg0u0td8.jpg)
    * Source files配置：所构建要上传的war包所在的位置，Jenkins workspace的相对路径!如构建后war包路径为：`/Users/xxx/.jenkins/workspace/xxx-prod/xxx-prod/target/xxx-prod-app.war`，那么Source files配置应为：`xxx-prod/target/xxx-prod-app.war`
    * Remove prefix：配置为要过滤war包前面的路径(如：`xxx-prod/target/`)，否则会在远程服务器上创建war包前面相应的文件夹
    * Remote directory：继承**系统设置SSH Servers**的`Remote Directory`，路径不存在则会创建！如例中配置，远程目录全路径则为：`/root/Jenkins-in`
    * Exec command：上传文件到远程服务器后运行到脚本文件
* 需要在应用服务器创建文件夹：Jenkins-in，在文件夹中复制一下脚本内容：xxx.sh
```shell
#!/bin/bash
#Time
log_time=`date +[%Y-%m-%d]%H:%M:%S`

###manual_properties###
tomcat_basehome=/webapp/tomcat-prod-8080
tomcat_port=8080
shell_environment=/bin/bash
war_Dir=/root/Jenkins-in
war_Name=xxx-app
war_package=${war_Name}.war
###manual_properties###


#update server environment
echo "**********************************  ${log_time} *************************************"
echo "~~~~~~~~~~~~~~updating server  environment start~~~~~~~~~~~~~~"
export JAVA_HOME=/usr/local/java
export JRE_HOME=/usr/local/java/jre
export PATH=$JAVA_HOME/bin:$PATH 
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar/
export CATALINA_2_HOME=${tomcat_basehome}
export CATALINA_2_BASE=${tomcat_basehome}
export TOMCAT_2_HOME=${tomcat_basehome}
sleep 3
echo "~~~~~~~~~~~~~~updating server  environment  end~~~~~~~~~~~~~~"

#build check funcation
echo "check tomcat status..."
check_tomcat_status(){
      netstat -ant|grep ${tomcat_port} > /dev/null 
      t=$?
       if [ $t -eq 0 ]; then
           echo "tomcat is running....port is ${tomcat_port}"
           echo "shutdown tomcat....."
           echo ">>>>>>>shutdown tomcat begin<<<<<<<<<"
            ${shell_environment} ${tomcat_basehome}/bin/shutdown.sh
           echo ">>>>>>>shutdown tomcat end<<<<<<<<<"
           sleep 5
       elif [ $t -ne 0 ];then 
             echo "tomcat is poweroff"
              ${shell_environment} ${tomcat_basehome}/bin/shutdown.sh
             sleep 5
       fi
} 

#check tomcat status invoke function
check_tomcat_status


#transfer  application package
deploy_Loaction=${tomcat_basehome}/webapps/
war_Dir_Data=`ls ${war_Dir}`
echo "--------------  begin  transfer  war package to tomcat webapps -------------------"

if [ -z $war_Dir ];then
     echo "Folder ${war_Dir} is empty.please check war package in this folder!"
     exit 1 
else
     echo "Find ${war_Dir} exist war package ${war_package}"
    # echo "deleteing old  package ${war_package} in ${war_Dir}"
    # rm ${war_Dir}/${war_package}
     echo "deleteing old  package ${war_package} in ${deploy_Loaction}" 
     rm -rf ${deploy_Loaction}${war_Name}*
     echo "start  transfer ${war_package} to ${deploy_Loaction}"
     mv ${war_Dir}/${war_package} ${deploy_Loaction}
     sleep 3 
fi
echo "--------------  transfer  war package to tomcat webapps  end -------------------"
#reboot tomcat 
echo "<<<<<<<<<rebooting  tomcat begin>>>>>>>>"
${shell_environment} ${tomcat_basehome}/bin/startup.sh
sleep 10
echo "<<<<<<<<<rebooting  tomcat end>>>>>>>>"
echo "the log you can read in canalina.out"
echo "************************ deploy war package into container Successlly  **********************************"
```
这段脚本意思就是shutdown旧应用，rebooting新应用！

# 参考文档

1. [jenkins部署Spring Boot](http://www.ityouknow.com/springboot/2017/11/11/springboot-jenkins.html)
2. [jenkins创建maven项目及ssh部署](https://www.cnblogs.com/parryyang/p/6256844.html)
3. [SSH: Transferred 0 file(s) 解决](https://www.cnblogs.com/zongyl/p/9157488.html)
4. [解决SSH: Transferred 0 file(s)](https://www.jianshu.com/p/ef6a4022b7b5)

