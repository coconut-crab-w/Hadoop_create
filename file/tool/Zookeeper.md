# Zookeeper 工具使用（日积月累）

## Zookeeper是干什么的
Zookeeper是一个分布式协调服务，用于实现分布式应用之间的同步。

### Zookeeper 安装时需要修改的配置文件
``` shell

# 心跳时间
tickTime=2000
# follow连接leader的初始化连接时间，表示tickTime的倍数
initLimit=10
# syncLimit配置表示leader与follower之间发送消息，请求和应答时间长度。
# 如果followe在设置的时间内不能与leader进行通信，那么此follower将被丢弃，tickTime的倍数
syncLimit=5
# 客户端连接超时时间ms
maxClientCnxns=600
# 客户端连接端口，访问 zookeeper的端口
clientPort=2181
# 节点数据存储及日志目录，需要提前创建
dataDir=/opt/module/apache-zookeeper-3.8.5-bin/data
dataLogDir=/opt/module/apache-zookeeper-3.8.5-bin/log
 
server.1=192.168.44.128:2888:3888
server.2=192.168.44.129:2888:3888
```