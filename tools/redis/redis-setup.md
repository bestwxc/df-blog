## redis安装教程
#### 安装信息
- 操作系统：redhat6.x(centos6.x)
- 安装包：redis-3.2.12.tar.gz（redis-5.0.0.tar.gz测试通过）
- 安装目录:/opt/software
- REDIS_HOME:/opt/software/redis-3.2.12,并建立软链接/opt/software/redis指向当前版本
- 配置文件目录：$REDIS_HOME/conf

#### 安装步骤 
- （可选）配置本地yum源
- 更新必要依赖
    + yum groupinstall Development tools
    + yum install gcc gcc-c++ gcc-g gcc-gnat gcc-java gcc-objc libgcj libgcj-devel libgnat libobjc libstdc++ zlib-devel zlib openssl openssl-devel
- 下载nginx安装包到/opt/install
- 解压安装包 cd /opt/install && tar -xvf redis-3.2.12.tar.gz
- cd redis-3.2.12
- make
- make install PREFIX=/opt/software/redis-3.2.12
- 建立软链接 ln -s /opt/software/redis-3.2.12 /opt/software/redis
- 建立配置文件目录并拷贝默认配置文件 mkdir -p /opt/software/redis/conf && cp /opt/install/redis-3.2.12/redis.conf /opt/software/redis/conf
- 修改默认配置文件，如下：
	+ 守护模式：daemonize的值，no --> yes
	+ 设置密码：取消“requirepass foobared”前的注释并设置为指定密码 ，如设置为123456:requirepass 123456
	+ 设置远程可连接：注释配置文件中的"bind 127.0.0.1"
- 启动redis /opt/software/redis/bin/redis-server /opt/software/redis/conf/redis.conf

#### 常用命令：
- 启动：/opt/software/redis/bin/redis-server /opt/software/redis/conf/redis.conf
- 停止：/opt/software/redis/bin/redis-cli -a 123456 shutdown
- 进入控制台： /opt/software/redis/bin/redis-cli -a 123456

#### 集群安装
**集群安装见[redis-cluster集群安装教程](https://blog.csdn.net/sinat_30133973/article/details/84165225)**
