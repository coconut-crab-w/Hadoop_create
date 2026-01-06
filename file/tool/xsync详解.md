## xsync详解

### 代码
```shell

#!/bin/bash

#1. 判断参数个数
if [ $# -lt 1 ]
then
    echo Not Enough Arguement!
    exit;
fi

#2. 遍历集群所有机器
for host in hadoop-dn1 hadoop-dn2 hadoop-dn3
do
    echo ====================  $host  ====================
    #3. 遍历所有目录，挨个发送

    for file in $@
    do
        #4. 判断文件是否存在
        if [ -e "$file" ]
            then
                #5. 获取父目录
                pdir=$(cd -P $(dirname $file); pwd)

                #6. 获取当前文件的名称
                fname=$(basename $file)
                ssh $host "mkdir -p $pdir"
                rsync -av $pdir/$fname $host:$pdir
            else
                echo $file does not exists!
        fi
    done
done
```
### 代码逐行注释


- #!/bin/bash
> 含义：指定脚本使用Bash解释器执行。这是Shell脚本的标准开头

- if [ $# -lt 1 ]到 exit
> 1. 含义：检查脚本参数个数是否小于1（即是否未提供参数）。如果参数不足，输出错误信息并退出脚本
> 2. 关键语法：
>    1. $#表示参数个数
>    2. -lt是数值比较运算符，表示"小于" 
>    3. exit终止脚本，默认退出码为0（成功），但这里未指定码值可能影响错误处理

- for host in hadoop-dn1 hadoop-dn2 hadoop-dn3
> 1. 含义：遍历主机列表（hadoop-dn1, hadoop-dn2, hadoop-dn3），对每台主机执行后续操作
> 2. 提示：主机名是硬编码的，实际应用中可改为从文件或变量读取

- echo ==================== $host ====================
> 含义：输出分隔符和当前主机名，便于日志阅读

- for file in $@
> 1. 含义：遍历所有传入的参数（即文件或目录路径），$@表示参数列表
> 2. 注意：如果参数包含空格或特殊字符，建议用引号包裹变量（如 "$@"）。
- if [ -e $file ]
> 1. 含义：检查当前文件或目录在本地是否存在。-e是文件测试运算符，用于验证路径存在性。
> 2. 风险：变量 $file未加引号，若路径含空格可能出错。建议改为 [ -e "$file" ]。

- pdir=$(cd -P $(dirname $file); pwd)
> 1. 含义：获取文件的绝对父目录路径。
>    1. 提示：dirname $file提取路径的目录部分（如 /home/user/dir来自 /home/user/dir/file.txt）。
>    2. cd -P切换到物理路径（解析符号链接），pwd输出绝对路径。
> 2. 示例：若 file是 ../src/data.txt，pdir可能是 /home/project/src。
- fname=$(basename $file)
> 含义：提取文件名或目录名（如 data.txt来自 /home/user/data.txt）
- ssh $host "mkdir -p $pdir"
> - 含义：通过SSH在远程主机上递归创建父目录（-p选项确保中间目录不存在时自动创建）
> - 注意：假设SSH已配置免密登录，否则需处理密码输入
- rsync -av $pdir/$fname $host:$pdir
> - 含义：使用 rsync同步文件到远程主机
>   1. -a归档模式，保留文件属性
>   2. -v显示详细输出
> - 风险：如果 $fname是目录，会递归同步子内容。若需排除某些文件，可添加 --exclude选项
- else echo $file does not exists!
> 含义：如果本地文件不存在，输出错误信息但继续处理其他文件（非致命错误）
- 循环结束标记
> done结束内层文件循环，外层的 done结束主机循环。