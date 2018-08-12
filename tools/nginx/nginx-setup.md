## nginx安装教程
#### 安装信息
- 操作系统：redhat6.x(centos6.x)
- 安装包：nginx-1.12.2.tar.gz（可支持nginx 1.9.x,1.10.x,1.11.x）
- 安装目录:/opt/software
- NGINX_HOME:/opt/software/nginx-1.12.2,并建立软连接/opt/software/nginx
- 配置文件目录：$NGINX_HOME/conf

#### 安装步骤
- 使用root用户登陆系统
- （可选）配置本地yum源
- 更新必要依赖
    + yum groupinstall Development tools
    + yum install gcc gcc-c++ gcc-g gcc-gnat gcc-java gcc-objc libgcj libgcj-devel libgnat libobjc libstdc++ zlib-devel zlib openssl openssl-devel
- 下载nginx_setup.zip 到/opt/install,下载地址：https://pan.baidu.com/s/1cplmzeoS0U3eEz7dHw9eMg，提取码：nuwg
- 进入/opt/install并解压nginx_setup.zip cd /opt/install && unzip nginx_setup.zip
- 进入nginx_setup
- 执行setup.sh bash setup.sh
- 执行完成后，进入/opt/software,可以看到nginx安装完成
- 建立软连接 ln -s /opt/software/nginx-1.12.2 /opt/software/nginx
- 启动/停止命令
    + 启动 service nginx start
    + 停止 service nginx stop
    + 重启 service nginx restart
    + 查看状态 service nginx status

#### 安装其他版本
本安装脚本支持其余nginx 版本，只需要修改setup.sh，并替换安装包即可。下面是安装脚本内容：
```
#!/bin/bash
nginx_version=nginx-1.12.2
cur_dir=`pwd`
tar -xvf pcre-8.10.tar.gz
cd pcre-8.10
./configure
make
make install
cd $cur_dir
tar -xvf openssl-1.0.2m.tar.gz
tar -xvf $nginx_version.tar.gz
cd $nginx_version
./configure --prefix=/opt/software/$nginx_version --with-http_stub_status_module --with-stream  --with-http_ssl_module --with-http_realip_module --add-module=$cur_dir/nginx-http-concat-master --with-openssl=$cur_dir/openssl-1.0.2m
#./configure --prefix=/opt/software/$nginx_version --with-http_stub_status_module --with-stream  --with-http_ssl_module --with-http_realip_module --with-openssl=$cur_dir/openssl-1.0.2m
make
make install
\cp -r $cur_dir/nginx /etc/init.d
chmod u+x /etc/init.d/nginx
chkconfig --level 2345 nginx on
```