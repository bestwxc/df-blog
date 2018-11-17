## redis-cluster集群安装教程
#### 安装信息
- 操作系统：redhat6.x(centos6.x)
- 安装包：redis-5.0.0.tar.gz
- 安装目录:/opt/software
- REDIS_HOME:/opt/software/redis-5.0.0,并建立软链接/opt/software/redis指向当前版本
- redis-cluster节点目录:/opt/software/redis-cluster/节点名称，本文使用端口库作为节点名称

#### 注意事项
- 每个redis节点使用两个TCP端口，一个是提供服务的端口，另一个是集群通信的端口，两者偏移10000.如果节点端口为7000，则通信端口为17000。
- redis集群不支持NAT或地址转换的环境。当获取的数据不在当前分片时，节点会返回一个重定向到另一个节点的指令，由客户端负责去连接指定节点获取数据。NAT环境外网地址和内网地址不一致，重定向返回的是转换前的地址，客户端无法连接。**同理，如果配置节点时使用的地址是127.0.0.1，其余机器也无法连接**。

#### redus-cluster目录下
- startAll.sh 启动本目录下所有节点
- stopAll.sh 停止本目录下所有节点
- createClush.sh 用于创建集群的命令，不能同步执行
- 各节点目录

#### 每个节点目录下，目录文件结构如下：
- conf: 配置文件目录
  - redis.conf 基础配置文件
- bin: 公共脚本目录
  - common.sh 公共脚本
- data: 数据文件目录
  - dump.rdb 数据文件
  - node-端口.conf 集群配置文件，由集群自动生成
- logs: 日志目录
  - redis.log redis日志
- start.sh 启动节点脚本
- stop.sh 停止脚本
- cli.sh 简单的控制命令脚本

#### 安装步骤
- 安装redis-5.0.0,详细步骤见博文[redis安装教程](https://blog.csdn.net/sinat_30133973/article/details/81612521)
- 配置redis集群
  - 建立redis集群目录： mkdir -p /opt/software/redis-cluster
  - 进入目录： cd /opt/software/redis-cluster
  - 下载节点配置放在reds-cluster目录,[下载地址](http://www.menghuanhua.com/df-blog/tools/redis/redis-cluster.zip)
  - 将redis-cluster.zip上传到/opt/software/redis-cluster并解压： unzip redis-cluster.zip
  - 根据情况增删节点目录，一般情况下
    - 单机测试模式： 同一个机器上，7000-7005共6各节点，三个分片，每个分片一个从节点
    - 基础集群模式：三个机器，每个机器上7000，7001两个节点，共六个节点，同样为三个分片，每个分片1个从节点
  - 启动所有节点： sh startAll.sh
  - 如果是多机集群，将所有机器上部署上节点并启动
  - 修改createCluter.sh中的节点IP和端口
  - 执行createCluter.sh，配置集群，如下是我所有的输出
```
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 172.18.16.237:7003 to 172.18.16.237:7000
Adding replica 172.18.16.237:7004 to 172.18.16.237:7001
Adding replica 172.18.16.237:7005 to 172.18.16.237:7002
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 4c975720cb9e54b300c7a133d0dda9756e847d0a 172.18.16.237:7000
   slots:[0-5460] (5461 slots) master
M: 1a55fdffc21b3787d773e14d6a8b097ca5296568 172.18.16.237:7001
   slots:[5461-10922] (5462 slots) master
M: 9c19c4cc004a5541757aad0afe97ed5397452e3c 172.18.16.237:7002
   slots:[10923-16383] (5461 slots) master
S: 2eb825ac53e012d51b66b6c38befe92d0c2ee09a 172.18.16.237:7003
   replicates 9c19c4cc004a5541757aad0afe97ed5397452e3c
S: 7a1c253e34ddc3bb18cddb374b12ec6d7cc57068 172.18.16.237:7004
   replicates 4c975720cb9e54b300c7a133d0dda9756e847d0a
S: 181dcdf4cb553759cafa9684e717a31b8b7082fa 172.18.16.237:7005
   replicates 1a55fdffc21b3787d773e14d6a8b097ca5296568
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
....
>>> Performing Cluster Check (using node 172.18.16.237:7000)
M: 4c975720cb9e54b300c7a133d0dda9756e847d0a 172.18.16.237:7000
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 181dcdf4cb553759cafa9684e717a31b8b7082fa 172.18.16.237:7005
   slots: (0 slots) slave
   replicates 1a55fdffc21b3787d773e14d6a8b097ca5296568
M: 1a55fdffc21b3787d773e14d6a8b097ca5296568 172.18.16.237:7001
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 7a1c253e34ddc3bb18cddb374b12ec6d7cc57068 172.18.16.237:7004
   slots: (0 slots) slave
   replicates 4c975720cb9e54b300c7a133d0dda9756e847d0a
S: 2eb825ac53e012d51b66b6c38befe92d0c2ee09a 172.18.16.237:7003
   slots: (0 slots) slave
   replicates 9c19c4cc004a5541757aad0afe97ed5397452e3c
M: 9c19c4cc004a5541757aad0afe97ed5397452e3c 172.18.16.237:7002
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

#### 常见问题
- 可以使用ps -ef|grep redis查看节点是否正常启动
```
[root@kfb_db237 redis-cluster]# ps -ef|grep redis
root     22029     1  0 08:21 ?        00:00:00 /opt/software/redis/bin/redis-server *:7002 [cluster]                                
root     22035     1  0 08:21 ?        00:00:00 /opt/software/redis/bin/redis-server *:7000 [cluster]                                
root     22041     1  0 08:21 ?        00:00:00 /opt/software/redis/bin/redis-server *:7005 [cluster]                                
root     22047     1  0 08:21 ?        00:00:00 /opt/software/redis/bin/redis-server *:7003 [cluster]                                
root     22053     1  0 08:21 ?        00:00:00 /opt/software/redis/bin/redis-server *:7004 [cluster]                                
root     22059     1  0 08:21 ?        00:00:00 /opt/software/redis/bin/redis-server *:7001 [cluster]                                
root     22938 17635  0 08:26 pts/0    00:00:00 grep redis
```

- 如果节点未正常启动，一般为下列两个问题
  - 内存不够，请更改redis内存配置，或者调整机器内存
  - 端口占用，除了通信占用外，还可能是集群端口被占用
- 当节点中存在数据时，无法创建集群。
- 当节点已经配置过后，无法再使用创建命令
- 本例中，可以使用下面命令快速清理集群数据和配置
  - cd /opt/software/redis-cluster
  - find . -name "node*" -exec rm -rf {} \;
  - find . -name "dump*" -exec rm -rf {} \;



