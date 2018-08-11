## nexux安装及配置Maven、pip、npm私服
#### 安装信息
- 操作系统 RedHat6.x(Centos6.x)
- nexus-oss版本 nexus-3.13.0-01-unix.tar.gz 
- JDK版本：JDK8
- 安装目录 /opt/software/nexus
- 应用目录 /opt/software/nexus/nexus-3.13.0-01
- 应用目录软连接 /opt/software/nexus/nexus
- 组件目录 /opt/software/nexus/sonatype-work
- 端口：8081（默认）
- nexus默认管理用户:admin/admin123
#### 常用操作
/opt/software/nexus/nexus/bin/nexus {start|stop|run|run-redirect|status|restart|force-reload}
#### 安装步骤
- 服务器参数调整，关闭防火墙等，本文略
- 安装JDK8，本文略
- 安装nexus
    + 创建安装目录及下载目录  mkdir -p /opt/{software,install}
    + 下载Nexus Repository Manager OSS 3.x 社区版，下载地址：https://www.sonatype.com/download-oss-sonatype，并将下载文件保存到/opt/install目录
    + 创建nexus目录  mkdir -p /opt/software/nexus
    + 解压压缩包到目录 tar -xvf /opt/install/nexus-3.13.0-01-unix.tar.gz  -C /opt/software/nexus
    + 创建应用目录软链接，方便版本更新  ln -s /opt/software/nexus/nexus-3.13.0-01 /opt/software/nexus/
    + 启动nexus  /opt/software/nexus/nexus/bin/nexus start
    + 使用浏览器访问 http://127.0.0.1:8081，进入管理平台，安装完成。
- （可选）修改配置文件，并重启。配置文件如下：
```
    -Xms1200M
    -Xmx1200M
    -XX:MaxDirectMemorySize=2G
    -XX:+UnlockDiagnosticVMOptions
    -XX:+UnsyncloadClass
    -XX:+LogVMOutput
    -XX:LogFile=../sonatype-work/nexus3/log/jvm.log
    -XX:-OmitStackTraceInFastThrow
    -Djava.net.preferIPv4Stack=true
    -Dkaraf.home=.
    -Dkaraf.base=.
    -Dkaraf.etc=etc/karaf
    -Djava.util.logging.config.file=etc/karaf/java.util.logging.properties
    -Dkaraf.data=../sonatype-work/nexus3
    -Djava.io.tmpdir=../sonatype-work/nexus3/tmp
    -Dkaraf.startLocalConsole=false
```
- 进入管理平台，使用默认密码admin/admin123登录，并修改管理用户密码,如图
        ![](images/nexus-change-password.png)
#### 配置Maven私服
- 打开管理平台，并使用管理用户登录。
- 进入管理页面，并进入仓库管理，对仓库进行增删。新装的nexus默认仓库如图。
        ![](images/nexus-maven-config-1.png)
- 配置仓库
一般情况下，maven私服的功能如下：
    + 代理中央仓库，默认的maven-central即中央仓库
    + 代理常用开源组件库，如spring（发布库/里程碑库/快照库）仓库、Oracle仓库
    + 建立私有仓库，分版本库和快照库，即默认的maven-releases和maven-snapshots
    + 建立third-party库，用于上传中央仓库没有，第三方的组件
    + 建立仓库组，便于配置使用，即默认的maven-public
默认的maven-central,maven-releases,maven-snapshots,maven-public可以直接使用，如果需要自定义名称，可以删除重新添加。本文仅保留maven-central，其余删除并重新添加，并分别命名为df-releases,df-snapshots,df-public。按照下表关键属性添加仓库：
    
        | 仓库名称 | 格式 | 类型 | 发布/快照库 | 是否允许重新发布 | 代理地址 |
        |:----------:|:-------:|:-----:|:--------:|:-------:|:----------|
        |maven-central|maven2|proxy|release|-|https://repo1.maven.org/maven2/|
        |df-releases|maven2|hosted|release|disabled redeploy|-|
        |df-snapshots|maven2|hosted|snapshot|allow redeploy|-|
        |third-party|maven2|hosted|release|disabled redeploy|-|
        |df-public|maven2|-|-|-|-|
        |spring-snapshots|maven2|proxy|snapshot|-|https://repo.spring.io/libs-snapshot-local|
        |spring-milestones|maven2|proxy|release|-|https://repo.spring.io/libs-milestone-local|
        |spring-releases|maven2|proxy|release|-|https://repo.spring.io/release|
        |maven.oracle.com|maven2|proxy|release|-|https://maven.oracle.com|

- 由于oracle仓库采用了认证，在按照上表特性进行配置后，需要进行特殊配置，配置方式如下。
    + 在oracle官网注册账户，注册地址：https://profile.oracle.com/myprofile/account/create-account.jspx
    + 在列表中，点击maven.oracle.com对应行，进入仓库配置。勾选如图的选项，并填入用户名和密码。
        ![](images/nexus-maven-config-2.png)
- 进入df-public，将其余hosted和proxy类型的仓库加入组中，如图：
        ![](images/nexus-maven-config-3.png)
- 配置好的仓库列表如图
        ![](images/nexus-maven-config-4.png)

