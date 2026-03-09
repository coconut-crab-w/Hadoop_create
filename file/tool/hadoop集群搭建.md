## Hadoop 集群搭建

### 1.集群部署规划

>1. NameNode和SecondaryNameNode不要安装在同一台服务器
>2. ResourceManager也很消耗内存，不要和NameNode、SecondaryNameNode配置在同一台机器上。

### 2. 配置文件

配置文件
. 默认配置文件 （文件存放在Hadoop的jar包中的位置）
. 自定义配置文件 （$HADOOP_HOME/etc/hadoop）
. 配置的文件：core-site.xml、hdfs-site.xml、yarn-site.xml、mapred-site.xml


### 3. 配置集群
#### 端口：
> 8020: NameNode 和 DataNode的rpc通讯端口，也是集群默认主机的地址
> 9870: web 访问端口
> 9868: 2nn的web访问端口
1. core-site.xml
``` shell 
    <!-- 指定NameNode的地址 这是 NameNode 接收客户端和 DataNode 元数据操作请求的默认端口-->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop-nn2:8020</value>
    </property>

    <!-- 指定hadoop数据的存储目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/module/hadoop-3.1.3/data</value>
    </property>

    <!-- 配置HDFS网页登录使用的静态用户为atguigu -->
    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>hadoop</value>
    </property>
```
2. hdfs-site.xml
``` shell
	<!-- nn web端访问地址-->
	<property>
        <name>dfs.namenode.http-address</name>
        <value>hadoop-nn2:9870</value>
    </property>
	<!-- 2nn web端访问地址-->
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>hadoop-dn3:9868</value>
    </property>
```

3. yarn-site.xml
``` shell
    <!-- 指定MR走shuffle -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

    <!-- 指定ResourceManager的地址-->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop-dn1</value>
    </property>

    <!-- 环境变量的继承 -->
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME
        </value>
    </property>
```

4. mapred-site.xml
``` shell
	<!-- 指定MapReduce程序运行在Yarn上 -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
```

### 群起集群
1. 配置workers
``` shell
 vim /opt/module/hadoop-3.1.3/etc/hadoop/workers

 hadoop-nn2
 hadoop-dn1
 hadoop-dn3
```
*注意：该文件中添加的内容结尾不允许有空格，文件中不允许有空行。*

2. 启动集群
> 第一次启动集群需要在NameNode节点初始化NameNode，格式化之前要先停止集群，然后删除集群上所有的data和logs目录，然后在进行初始化。初始化命令:`hdfs namenode -format`   

集群的启动命令和yarn的启动命令
> hdfs的启动命令 ：`sbin/start-dfs.sh`,yarn的启动命令一定要在ResourceManager的节点上启动，命令：`sbin/start-yarn.sh`

Web端查看HDFS的NameNode
- 浏览器上输入：http://101.35.50.136:9870/
- 查看HDFS上存储的数据信息

Web端查看YARN的ResourceManager
- 浏览器上输入：http://124.223.112.235:8088/
- 查看YARN上运行的Job信息

3. 集群的基本测试



4. 配置历史服务器
为了查看程序的历史运行情况，需要配置一下历史服务器。
> 配置mapred-site.xml
``` shell
<!-- 历史服务器端地址 -->
<property>
    <name>mapreduce.jobhistory.address</name>
    <value>hadoop-nn2:10020</value>
</property>

<!-- 历史服务器web端地址 -->
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop-nn2:19888</value>
</property>
```

5. 配置日志的聚集
日志聚集概念：应用运行完成以后，将程序运行日志信息上传到HDFS系统上。

