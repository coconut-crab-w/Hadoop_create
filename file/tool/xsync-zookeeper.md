## 编写一键启动zookeeper命令

```shell
#!/bin/bash

# 定义集群服务器列表
SERVERS=("hadoop-dn1" "hadoop-dn2" "hadoop-dn3")

# 定义ZooKeeper的安装路径（请根据实际安装路径修改）
ZK_HOME="/opt/module/apache-zookeeper-3.8.5-bin"

# 使用说明函数
usage() {
    echo "用法: $0 {start|stop|status}"
    echo "  start   - 启动集群中所有服务器的ZooKeeper服务"
    echo "  stop    - 停止集群中所有服务器的ZooKeeper服务"
    echo "  status  - 检查集群中所有服务器的ZooKeeper服务状态"
    exit 1
}

# 检查参数个数
if [ $# -ne 1 ]; then
    usage
fi

# 启动ZooKeeper服务函数
start_zookeeper() {
    local server=$1
    echo "========== 在 $server 上启动ZooKeeper服务 =========="
    ssh "$server" "source /etc/profile; $ZK_HOME/bin/zkServer.sh start"
    if [ $? -eq 0 ]; then
        echo "✓ $server - ZooKeeper启动命令执行成功"
    else
        echo "✗ $server - ZooKeeper启动失败"
    fi
    echo
}

# 停止ZooKeeper服务函数
stop_zookeeper() {
    local server=$1
    echo "========== 在 $server 上停止ZooKeeper服务 =========="
    ssh "$server" "source /etc/profile; $ZK_HOME/bin/zkServer.sh stop"
    if [ $? -eq 0 ]; then
        echo "✓ $server - ZooKeeper停止命令执行成功"
    else
        echo "✗ $server - ZooKeeper停止失败"
    fi
    echo
}

# 检查ZooKeeper状态函数
check_zookeeper_status() {
    local server=$1
    echo "========== 检查 $server 的ZooKeeper服务状态 =========="
    ssh "$server" "source /etc/profile; $ZK_HOME/bin/zkServer.sh status"
    local status_code=$?
    if [ $status_code -eq 0 ]; then
        echo "✓ $server - ZooKeeper服务运行正常"
    else
        echo "✗ $server - ZooKeeper服务异常或未运行"
    fi
    echo
}

# 根据传入参数执行相应操作
case "$1" in
    "start")
        echo "开始启动ZooKeeper集群..."
        for server in "${SERVERS[@]}"; do
            start_zookeeper "$server"
            # 添加短暂延迟，避免同时启动造成冲击
            sleep 2
        done
        echo "ZooKeeper集群启动命令执行完成"
        ;;
        
    "stop")
        echo "开始停止ZooKeeper集群..."
        for server in "${SERVERS[@]}"; do
            stop_zookeeper "$server"
            sleep 1
        done
        echo "ZooKeeper集群停止命令执行完成"
        ;;
        
    "status")
        echo "开始检查ZooKeeper集群状态..."
        for server in "${SERVERS[@]}"; do
            check_zookeeper_status "$server"
        done
        echo "ZooKeeper集群状态检查完成"
        ;;
        
    *)
        usage
        ;;
esac

# 显示最终结果摘要
echo "=================================================="
echo "操作完成时间: $(date)"
echo "=================================================="
```