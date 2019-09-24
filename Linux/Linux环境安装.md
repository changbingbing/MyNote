​	Linux环境

1. [安装文档](https://jingyan.baidu.com/article/4dc4084868a1e4c8d946f133.html)

2. [Centos国内镜像](http://centos.ustc.edu.cn/centos/)、[官网下载](http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso)

3. root/root0327、changbingbing/root0327

4. [centos7之系统优化方案](https://www.cnblogs.com/jokerbj/p/9133093.html) 

5. 固定IP设置：less /etc/sysconfig/network-scripts/ifcfg-enp0s3

   ```she
   BOOTPROTO=static
   
   ONBOOT=yes
   
   IPADDR=192.168.36.106
   NETMASK=255.255.248.0
   GATEWAY=192.168.32.1
   ```

6. vi /etc/resolv.conf

   ```shell
   #修改DNS信息
   nameserver 114.114.114.114#国内移动、电信和联通通用的DNS
   nameserver 8.8.8.8#GOOGLE公司提供的DNS，该地址是全球通用的
   search localdomain
   
   service network restart   #重启网卡，生效
   ```

7. 查看路由表

   ```she
   netstat -rn
   ```

8. 增加网关

   ```properties
   route add default gw 192.168.175.1
   ```

9. 关闭防火墙

  ```shell
  service iptables stop
  ```

  1. 若提示"Redirecting to /bin/systemctl stop iptables.service"，则：

    1. **使用systemctl**

      ```shell
      systemctl [start|stop|restart|save|status] iptables.service
      ```

    2. **安装iptables-services**
      切换到root用户下，执行：

      ```shell
      yum install iptables-services
      systemctl enable iptables.service //设置开机启动
      ```

      之后就可以使用以下指令了：

      ```shell
      service iptables [start|stop|restart|save|status]
      ```

