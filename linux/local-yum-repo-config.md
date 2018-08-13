## 利用ISO镜像文件搭建本地yum源教程
#### 安装信息
- 操作系统：redhat6.x(centos6.x)
- 系统镜像包：rhel-server-6.9-x86_64-dvd.iso
- 镜像挂载（解压）目录：/data/yum

#### 安装步骤
- 上传镜像到目标服务器的/opt/install目录
- 创建挂载目录：  mkdir -p /data/yum
- 挂载镜像：mount -o loop /opt/install/rhel-server-6.9-x86_64-dvd.iso  /data/yum/
- 备份原yum配置文件： cp -r /etc/yum.repos.d /etc/yum.repos.d.bak
- 增加自定义本地yum源的配置，一般一个配置一个文件。本例中配置文件名称为rhel-source.repo,配置文件内容如下：
```
    [rhel-source]
    name=Red Hat Enterprise Linux $releasever - $basearch - Source
    baseurl=file:///data/yum
    enabled=1
    gpgcheck=0
    gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```

- 清理索引缓存： yum clean all
- 重建索引： yum list
- 使用yum命令测试，如果正常安装包，则配置成功。