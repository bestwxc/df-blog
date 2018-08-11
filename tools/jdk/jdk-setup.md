## JDK安装
#### 安装信息
- Linux
    + 操作系统: redhat6.x(centos6.x)
    + 安装文件: jdk-8u181-linux-x64.tar.gz
    + 安装目录: /usr/local/java
- Window
    + 操作系统: win10
    + 安装文件: jdk-8u181-windows-x64.exe
    + 安装目录: D:\develop\JDK

#### Linux安装步骤
- 创建安装文件保存目录 mkdir -p /opt/install
- 创建安装目录 mkdir -p /usr/local/java
- 从官网下载安装文件jdk-8u181-linux-x64.tar.gz，并保存在/opt/install
- 解压到安装目录 tar -xvf /opt/install/jdk-8u181-linux-x64.tar.gz -C /usr/local/java
- 创建软链接，当系统存在多版本jdk时，软链接指向默认版本 ln -s /usr/local/java/jdk1.8.0_181 /usr/local/java/jdk
- 在 /etc/profile 中添加下列内容
```
    export   JAVA_HOME=/usr/local/java/jdk
    export   JRE_HOME=$JAVA_HOME/jre
    export   PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
    export   CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib:$JRE_HOME/lib
```

- 执行 source /etc/profile 使环境变量生效
- 使用 java -version 验证安装。如果正常输入版本信息则安装完成。

#### Window 安装步骤
- 下载安装文件jdk-8u181-windows-x64.exe
- 双击安装，一路next安装向导指定安装目录，并取消jre的安装。如图：
    ![](http://www.menghuanhua.com/df-blog/tools/jdk/images/jdk-setup-win-1.png)
- 配置环境变量
    + 配置JAVA_HOME=D:\develop\JDK\jdk1.8.0_181
    + 在PATH前面加入 JAVA_HOME\bin
- 快捷键win+R打开运行对话框，输入cmd并回车打开控制台。在控制台中使用java -version，正常打印版本信息则安装完成。
