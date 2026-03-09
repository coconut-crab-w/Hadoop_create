### JDK 安装
1. 下载jdk包：wget https://repo.huaweicloud.com/java/jdk/8u151-b12/jdk-8u151-linux-x64.tar.gz
2. 创建目录:`mkdir /opt/module`,`mkdir /opt/software`
3. 解压jdk包命令：`tar -zvxf jdk-8u151-linux-x64.tar.gz -C /opt/module/`
4. 配置jdk环境：①新建`vim /etc/profile.d/my_env.sh` 
``` shell
export JAVA_HOME=/opt/module/jdk1.8.0_151
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```
5. 配置启动命令：`source /etc/profile`
6. host 文件配置：总而言之，对于自己机器上的ip映射就填自己的内网ip，自己机器上对其他ip的映射就是他们的公网ip
7. 修改hostname文件，这个文件修改后，需要reboot重启


### 创建用户
> 1. 创建hadoop用户：`useradd hadoop`
> 2. 设置hadoop用户密码：`passwd hadoop`  `密码：Had@wang@1995`
> 3. 配置hadoop用户具有root权限，方便后期hadoop用户在输入命令时，加上sudo 执行root命令:
>    1.`vim /etc/sudoers`  在 %wheel 的下一行添

### 配置ssh免密登录

 1. 生成密钥的地址: `/home/hadoop/.ssh`
 2. 发起公钥请求：`ssh-keygen -t rsa`
 3. 生成了两个文件：`id_rsa`(私钥)和`id_rsa.pub`(公钥)
 4. 将公钥文件拷贝到要免密登录的服务器中：`ssh-copy-id hadoop-nn2`


 ### 编写xsync脚本
 >    1. xsync 集群分发脚本如果期望在任何路劲都能执行，那么脚本需要放在声明了全局变量的路径下。
 >    ![img_2.png](img_2.png)
 >    2. 脚本实现：
 >       1. `cd /home/hadoop`
 >       2. `mkdir bin`
 >       3. `cd bin`
 >       4. `vim xsync`(编写代码在《xsync详解.md》)

 ### 安装Hadoop集群
 1. 下载hadoop安装包：wget https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-3.1.3/hadoop-3.1.3.tar.gz
 2. 安装：`tar -zxvf [包名] -C [安装路径]`
 3. 配置环境变量：
 ``` shell
export JAVA_HOME=/opt/module/jdk1.8.0_151
export HADOOP_HOME=/opt/module/hadoop-3.1.3
export PATH=$JAVA_HOME/bin:$HADOOP_HOME/bin:$PATH
export PATH=$PATH:$HADOOP_HOME/sbin
export HADOOP_LOG_DIR=/opt/module/hadoop-3.1.3/logs
export YARN_LOG_DIR=$HADOOP_LOG_DIR
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
 ```

4. 创建文件夹
``` shell
mkdir /usr/hadoop/hadoop-3.1.3/tmp  [临时数据存储目录]
 
mkdir /usr/hadoop/hadoop-3.1.3/hdfs
 
mkdir /usr/hadoop/hadoop-3.1.3/hdfs/name 【hdfs的元数据存储位置】
 
mkdir /usr/hadoop/hadoop-3.1.3/hdfs/data 【hdfs的数据存储位置】
```

5. 配置文件
> core-site.xml
> hdfs-site.xml
> mapred-site.xml
> yarn-site.xml
> workers
> hadoop-env.sh、yarn-env.sh、mapred-env.sh 添加jdk安装位置 `export JAVA_HOME=/opt/module/jdk1.8.0_151`
- 进行 xsync 分发


### 启动Hadoop集群
1. 首次启动，要初始化NameNode : `hdfs namenode -format`
如果出现了SHUTDOWN_MSG: Shutting down NameNode at xxx信息也不要慌张，向上寻找信息，如果能找到 INFO common.Storage:Storage directory ******省略 has been successfully formatted信息，则格式化成功了，忽略后面的SHUTDOWN_MSG即可

### 编写启动文件
``` shell
#!/bin/bash
if [ $# -lt 1 ]
then
    echo "No Args Input..."
    exit ;
fi

case $1 in
"start")
        echo " =================== 启动 hadoop集群 ==================="

        echo " --------------- 启动 hdfs ---------------"
        ssh master "/opt/module/hadoop-3.1.3/sbin/start-dfs.sh"
        echo " --------------- 启动 yarn ---------------"
        ssh slave-dn1 "/opt/module/hadoop-3.1.3/sbin/start-yarn.sh"
        echo " --------------- 启动 historyserver ---------------"
        ssh master "/opt/module/hadoop-3.1.3/bin/mapred --daemon start historyserver"
;;
"stop")
        echo " =================== 关闭 hadoop集群 ==================="

        echo " --------------- 关闭 historyserver ---------------"
        ssh master "/opt/module/hadoop-3.1.3/bin/mapred --daemon stop historyserver"
        echo " --------------- 关闭 yarn ---------------"
        ssh slave-dn1 "/opt/module/hadoop-3.1.3/sbin/stop-yarn.sh"
        echo " --------------- 关闭 hdfs ---------------"
        ssh master "/opt/module/hadoop-3.1.3/sbin/stop-dfs.sh"
;;
*)
    echo "Input Args Error..."
;;
esac
 ```


