### spark 的下载
- 版本兼容问题:
> 使用的hadoop-3.1.3的版本与Spark3.0.x版本适配

- 下载地址
`wget https://mirrors.huaweicloud.com/apache/spark/spark-3.3.1/spark-3.3.1-bin-hadoop3.tgz`

- 解压
`tar -zvxf spark-3.3.1-bin-hadoop3.tgz -C /opt/module`

- 案例求PI
`./spark-submit --class org.apache.spark.examples.SparkPi --master local[2] ../examples/jars/spark-examples_2.12-3.3.1.jar 10`

- 监控界面
> http://master:4040 (任务正在执行中的时候才能在这个网页看见)


- Spark on Yarn 的方式进行部署

- 按需修改（生产上不需要修改）
>  修改 hadoop 配置文件 /opt/module/hadoop/etc/hadoop/yarn-site.xml, 添加如下内容
>  因为测试环境虚拟机内存较少，防止执行过程进行被意外杀死，做如下配置
``` xml
<!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认是 true-->
<property>
    <name>yarn.nodemanager.pmem-check-enabled</name>
    <value>false</value>
</property>

<!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认是 true-->
<property>
    <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
</property>
```

*修改Spark的配置文件*
> 文件位置：`/opt/module/spark/conf/spark-env.sh.template`
> 命令： `cp spark-env.sh.template spark-env.sh`
> 修改内容：在文件的最后添加`YARN_CONF_DIR=/opt/module/hadoop-3.1.3/etc/hadoop`

- 案例求PI
`./spark-submit --class org.apache.spark.examples.SparkPi --master yarn --driver-memory 2g --executor-memory 2g --num-executors 2 ../examples/jars/spark-examples_2.12-3.3.1.jar 10`