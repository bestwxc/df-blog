# ffmpeg 4.0.2 编译步骤
### 编译信息
- 系统版本： redhat6.6(CentOS6.6)
- ffmpeng版本： ffmpeg
- 编译参数： --enable-shared --enable-libxavs --enable-libx264 --enable-libx265 --enable-libvpx --enable-libfdk-aac --enable-libcodec2 --enable-libmp3lame --enable-libtwolame --enable-gpl --enable-nonfree --disable-x86asm

## 编译步骤
### 安装基础的依赖包（一般情况下可以跳过）
- yum groupinstall Development tools
- yum install gcc gcc-c++ gcc-g gcc-gnat gcc-java gcc-objc libgcj libgcj-devel libgnat libobjc libstdc++ zlib-devel zlib openssl openssl-devel
- yum install cmake

### 将所有安装包拷贝到目标机器的/opt/install/ffmpeg
- mkdir -p /opt/install/ffmpeg
- 将所有包上传到/opt/install/ffmpeg

### 编译视频编码解码包
#### libxavs
- cd /opt/install/ffmpeg
- unzip xavs-code-r55-trunk.zip
- cd xavs-code-r55-trunk
- ./configure --enable-shared --disable-asm
- make && make install

#### libx264
- cd /opt/install/ffmpeg
- unzip x264.zip
- cd x264
- ./configure --enable-shared --disable-asm
- make && make install

#### libx265
- yum install cmake (可选)
- cd /opt/install/ffmpeg
- tar -xvf x265_2.9.tar.gz
- cd x265_2.9/build/linux
- bash make-Makefiles.bash (执行完，停止时按q)
- make && make install

#### libvpx
- cd /opt/install/ffmpeg
- rpm -Uvh nasm-2.09-1.x86_64.rpm(可选，有版本要求)
- unzip libvpx-1.7.0.zip
- cd libvpx-1.7.0
- ./configure --enable-shared
- make && make install

#### libfdk-aac
- cd /opt/install/ffmpeg
- tar -xvf fdk-aac-0.1.6.tar.gz
- cd fdk-aac-0.1.6
- ./configure --enable-shared
- make && make install

#### libcodec2
- cd /opt/install/ffmpeg
- tar -xvf codec2-0.4.1.tar.gz
- cd codec2-0.4.1
- mkdir build_linux
- cd build_linux/
- cmake ..
- make && make install

#### libmp3lame
- cd /opt/install/ffmpeg
- tar -xvf lame-3.100.tar.gz
- cd lame-3.100
- ./configure --enable-shared
- make && make install

#### libtwolame
- cd /opt/install/ffmpeg
- tar -xvf twolame-0.3.13.tar.gz
- cd twolame-0.3.13
- ./configure --enable-shared
- make && make install

### 编译安装ffmpeg
- cd /opt/install/ffmpeg
- tar -xvf ffmpeg-4.0.2.tar.bz2
- cd ffmpeg-4.0.2
- export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH
- export   LD_LIBRARY_PATH=./:/usr/local/lib:/usr/local/lib64:$LD_LIBRARY_PATH
- ./configure --enable-shared --enable-libxavs --enable-libx264 --enable-libx265 --enable-libvpx --enable-libfdk-aac --enable-libcodec2 --enable-libmp3lame --enable-libtwolame --enable-gpl --enable-nonfree --disable-x86asm
- make && make install

### 检查环境变量设置，如果/etc/profile中没有以下配置，则加入
- export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH
- export   LD_LIBRARY_PATH=./:/usr/local/lib:/usr/local/lib64:$LD_LIBRARY_PATH