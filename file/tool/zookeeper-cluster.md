``` shell 
#!/bin/bash

for host in nnode2 nnode3 nnode4 //节点名称
do
        case $1 in
        "start"){
                echo "------------ $host zookeeper -----------"
                ssh $host "source /etc/profile; zkServer.sh start"
        };;
        "stop"){
                echo "------------ $host zookeeper -----------"
                ssh $host "source /etc/profile; zkServer.sh stop"
        };;
        "status"){
                echo "------------ $host zookeeper -----------"
                ssh $host "source /etc/profile; zkServer.sh status"
        };;
        esac
done
```