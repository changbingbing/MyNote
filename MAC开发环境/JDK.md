* [Mac系统配置JDK环境变量](https://www.cnblogs.com/cb0327/p/6670154.html)

* Mac JDK卸载

  * 输入：tar -xzvf /usr/tools/zookeeper-3.4.13.tar.gz -C /usr/apps/

    ```shell
    xssudo rm -fr /Library/Internet\ Plug-Ins/JavaAppletPlugin.plugin 
    sudo rm -fr /Library/PreferencesPanes/JavaControlPanel.prefpane
    ```

  * 查看当前版本：

    ```shell
    ls /Library/Java/JavaVirtualMachines/
    jdk-11.0.2.jdk	jdk1.8.0_74.jdk
    ```

  * 删除要卸载的版本即可：

    ```shell
    rm -rf jdk-11.0.2.jdk
    ```

