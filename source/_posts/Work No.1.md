---
title: Linux环境安装和配置
tags: Linux
date: 2024-07-09
categories: Linux
---

# Linux环境安装和配置

## 一、安装VMware（16/17）

安装版本：VMware® Workstation 16 Pro

安装保姆链接：[安装虚拟机（VMware）保姆级教程（附安装包）_vmware虚拟机-CSDN博客](https://blog.csdn.net/weixin_74195551/article/details/127288338)

破解链接：[VMware Workstation虚拟机合集+激活密钥_Win+Linux_10∕11∕12∕14∕15∕16∕17 - 『逆向资源区』 - 吾爱破解 - LCG - LSG |安卓破解|病毒分析|www.52pojie.cn](https://www.52pojie.cn/thread-1804571-1-1.html)

破解码：ZF3R0-FHED2-M80TY-8QYGC-NPKYF

MobaXtem链接：[VMware虚拟机配置、连接MobaXterm_mobaxtem怎么连接虚拟机-CSDN博客](https://blog.csdn.net/qq_42578036/article/details/107710339)

终极链接：[保姆级教程|VMware安装Ubuntu20.04(系统安装+网络配置+open-vm-tools安装+国内软件源更新)-CSDN博客](https://blog.csdn.net/lhl_blog/article/details/123406322)

报错：

VMware Workstation 无法连接到虚拟机

点开虚拟机“属性”，在“兼容性”里面设置为以管理员身份运行此程序。

VMware创建新的虚拟机，弹出“您已输入用户名，客户机操作将保留此用户名”错误提示

【root】被系统占用了，让用户新建一个低权限的账户，修改用户名，不要再使用【root】，如：改为【user】或【your_name】

VMware Ubuntu ping 百度不通

选择 虚拟机->设置->网络适配器->自定义特定虚拟网络->选择VMnet1(桥接网络)->确定

目的是确定我们使用的是VMnet1(桥接网络)，之后就可以ping通

安装磁盘管理工具gparted并运行

```
sudo apt install gparted
sudo gparted
```

报错是权限不足,那么修改挂载点的权限即可(注:所谓"挂载"的概念体现的是Linux"一切皆文件"的思想,物理世界中的一块硬盘在Linux系统的逻辑中也被映射为一个文件)

```
sudo mount -o remount -rw / 
sudo mount -o remount -rw /var/snap/firefox/common/host-hunspell
```

## 二、安装Ubuntu（22.04）

安装版本：Linux Ubuntu22.04.4

安装保姆链接：[ubuntu 22.04下载安装_ubuntu22.04下载-CSDN博客](https://blog.csdn.net/weixin_42640280/article/details/128351105)

清华源：[清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/)

步骤指令

```
sudo su //进入管理者模式

apt-get update //更新apt

apt-get install net-tools //安装ifconfig相关配件

apt-get install openssh-server //下载和安装ssh

service ssh restart //启动ssh
```

## 三、安装应用依赖第三方库

将工程导入虚拟机中，tar -xvf 解压工程包

git clone 的所有包的位置可以随机放置，但是需要对每个包进行编译，编译参照readme文件或者网上查找，如果下载不下来，可以直接去官网直接下载zip格式，使用unzip xxx.zip进行安装

```
sudo apt-get install -y build-essential libsdl2-dev

git clone http://172.23.88.26:3333/zhangyufeng/libcrlog.git  //按照readme文档可以编译

sudo apt-get install  libjson-c-dev

git clone https://github.com/protocolbuffers/protobuf.git
	//没安装成功，bash: ./autogen.sh: No such file or directory
	//在晚上的时候安装成功了，原因是高版本的缺失了./autogen.sh脚本，进行降版本就可以了，花费4小时
	//终极感谢博主：https://blog.csdn.net/jax_fanyang/article/details/135937002
	
    1.11 点击链接下载3.21.12版本源码。https://github.com/protocolbuffers/protobuf/releases/tag/v2
    1.12 cd protobuf-21.12/
    1.13 ./autogen.sh
    1.14 ./configure --prefix=/usr/local/protobuf
    1.15 make
    1.16 sudo make install
    1.17 sudo vim /etc/profile
    1.18 #添加以下内容：

    #(动态库搜索路径) 程序加载运⾏期间查找动态链接库时指定除了系统默认路径之外的其他路径
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/protobuf/lib/
    #(静态库搜索路径) 程序编译期间查找动态链接库时指定查找共享库的路径
    export LIBRARY_PATH=$LIBRARY_PATH:/usr/local/protobuf/lib/
    #执⾏程序搜索路径
    export PATH=$PATH:/usr/local/protobuf/bin/
    #c程序头⽂件搜索路径
    export C_INCLUDE_PATH=$C_INCLUDE_PATH:/usr/local/protobuf/include/
    #c++程序头⽂件搜索路径
    export CPLUS_INCLUDE_PATH=$CPLUS_INCLUDE_PATH:/usr/local/protobuf/include/
    #pkg-config 路径
    export PKG_CONFIG_PATH=/usr/local/protobuf/lib/pkgconfig/
    1.18 source /etc/profile
    1.19  protoc --version 
    执行之后应该看到==libprotoc 3.21.12==如果出现这个版本信息则安装成功
    
git clone https://github.com/protobuf-c/protobuf-c  //按照readme文档可以编译

git clone https://github.com/eclipse/paho.mqtt.c.git
	
	bash
    cd paho.mqtt.c
    mkdir build && cd build
    cmake ..
    make
    sudo make install

git clone https://github.com/eclipse/paho.mqtt.cpp.git（1.3.10）  //安装失败

git clone https://github.com/aliyun/aliyun-oss-cpp-sdk.git

    cd <path/to/aliyun-oss-cpp-sdk>
    mkdir build
    cd build
    cmake ..
    
    sudo apt-get install libcurl4-openssl-dev libssl-dev
    make

git clone https://github.com/open-source-parsers/jsoncpp.git

    mkdir -p build/debug
    cd build/debug
    cmake -DCMAKE_BUILD_TYPE=debug -DBUILD_STATIC_LIBS=ON -DBUILD_SHARED_LIBS=OFF -DARCHIVE_INSTALL_DIR=. -G "Unix Makefiles" ../..
    make

sudo apt install libapr1-dev

sudo apt install libaprutil1-dev

sudo apt install libmxml-dev

sudo apt-get install uuid-dev 

git clone https://github.com/aliyun/aliyun-oss-c-sdk.git //好像安装有点问题

    ./configure
    make
    make install

https://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/attach/102787/cn_zh/1558078831675/VodSDK-C_1.0.0.tar.gz?spm=a2c4g.11186623.0.0.65c71cd50vaal4&file=VodSDK-C_1.0.0.tar.gz //不知道怎么安装

//找阿里云帮忙实现：[使用C/C++ SDK上传文件_视频点播(VOD)-阿里云帮助中心 (aliyun.com)](https://help.aliyun.com/zh/vod/developer-reference/upload-sdk-for-c-or-cpp?spm=a2c4g.11186623.0.0.61dd2c16XPqrTn#multiTask7060)

    cmake .
    make
    make install

git clone https://github.com/ithewei/libhv

    mkdir build
    cd build
    cmake ..
    cmake --build .

sudo apt-get install libboost-all-dev

git clone https://github.com/inotify-tools/inotify-tools.git//安装不知道成没成功

sudo apt-get install libfreeimage3 libfreeimage-dev

git clone https://github.com/dpilger26/NumCpp.git

sudo apt-get install libavformat-dev

sudo apt-get install libswscale-dev
```

## 四、在PC上编译运行应用

make 编译程序

make clean 清除编译

头文件所在位置：

```
/usr/include/

/usr/local/include/

cp  -r /usr/local/include/vod_sdk/ /usr/include/vod_sdk/  //缺失upload.h头文件，将他进行复制到usr/include/文件夹当中

- 安装依赖
  sudo apt install libjson-c-dev

- 编译
  make

- 安装
  sudo make install

- 更新系统动态库
  sudo ldconfig
```

运行项目

先获得虚拟设备参数，导入config中"/home/user/creality/userdata/config/"

开始编译主程序  make clean && make -j4（j4、j8代表进程速度，越高越快）

获得各个文件的单独运行包，在vscode中运行./master-server，即可跑通实体机

linux中/opt目录用来**安装附加软件包**，是用户级的程序目录，可以理解为D:/Software。 安装到/opt目录下的程序，它所有的数据、库文件等等都是放在同个目录下面。 opt有可选的意思，这里可以用于放置第三方大型软件（或游戏），当你不需要时，直接rm -rf掉即可。

## 五、交叉编译到目标板运行

交叉编译讲解：[交叉编译入门及必要配置方法总结_gcc-manifest.txt-CSDN博客](https://blog.csdn.net/lc315yuhuofei/article/details/103782049)

虚拟机交叉编译教程：[立创泰山派学习05-虚拟机ubuntu安装交叉编译工具 - zbl1118 - 博客园 (cnblogs.com)](https://www.cnblogs.com/zblblog/p/18136017)

## 六、知识点总结

#### 1.交叉编译知识点

交叉编译的目的是在一台架构A主机平台上编译另一种架构B目标平台的二进制文件或者库，交叉编译在目标系统平台（开发出来的应用程序序所运行的平台）难以或不容易编译时非常有用。 完整的Linux编译环境需要很多支持包，交叉编译使我们不需要花时间将各种支持包移植到目标板上

主机平台：PC端 Windows 10 专业工作站版

目标平台：Linux Ubuntu22.04.4/VMware® Workstation 16 Pro

通常交叉编译工具链命名规则为：arch-core-kernel-system

arch：目标平台架构，如上文提到的arm，mips等；

core：有两种种情况，第一是CPU Core，如Cortex A8；第二是指定工具链的供应商。如果没有特殊指定，则留空不填。这一组命名比较灵活，有以厂家名称命名的，有以开发者命名的，也有以开发板命名的，或者直接是none或cross的；

kernel： 目标平台的OS，见过的有linux，uclinux，bare-metal（无OS）；

system：嵌入式应用二进制接口（Embedded Application Binary Interface），交叉编译工具链所选择的库函数和目标映像的规范，如gnu，gnueabi等。其中gnu等价于glibc+oabi；gnueabi等价于glibc+eabi。若不指定，则也可以留空不填；

上述命名规则并不是统一的规范，使用的时候作为参考就行。我使用的交叉编译工具链名称为：gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu。

获取交叉编译工具链两个途径：

直接下载知名厂家已经编译好的工具链。

```
https://www.linaro.org/downloads/
http://ftp.arm.linux.org.uk/pub/armlinux/toolchain/
http://www.denx.de/en/Software/WebHome
https://launchpad.net/gcc-arm-embedded
```

自己编译交叉编译工具链

编译交叉编译工具链的工具：crosstool-NG、Buildroot、Embedded Linux Development Kit (ELDK)

#### 2.Linux 系统编程知识点

[Linux 系统编程从入门到进阶 学习指南-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/1457993)

什么是库函数？

库函数是预编写的代码，存储在库文件中，供程序员使用。它们通过系统调用和操作系统的内核通信。例如，printf（） 是 C 语言的一个库函数，它内部使用 write（） 系统调用来和内核进行交互。

[Linux C函数库大全 - 一觉醒来写程序 - 博客园 (cnblogs.com)](https://www.cnblogs.com/realjimmy/p/12844359.html)

参考手册：[介紹 | Linux C API 参考手册 (gitbooks.io)](https://wizardforcel.gitbooks.io/linux-c-api-ref/content/index.html)

**进程究竟是什么？**

每当你启动一个程序，Linux 系统都会创建一个新的进程。这个进程有它自己的内存地址、系统资源和状态。简而言之，进程是程序的一个运行实例。

**1.管道 （Pipe）**

管道是 Linux 中用于进程间通信的一种机制。它们分为两种类型：**匿名管道**和**有名管道**。

**2.信号 (Signals)**

在 Linux 中，信号是一种用于进程间通信（IPC）的机制，允许操作系统或一个进程向另一个进程发送简单的消息。信号主要用于传递关于系统事件的通知，例如中断请求、程序异常、或其他重要事件。每个信号代表了一个特定类型的事件，并且进程可以根据收到的信号执行相应的动作。

信号是异步的，意味着它们可以在任何时间点被发送到进程，通常与进程的正常控制流无关。信号的使用为进程提供了一种处理外部事件和错误的方式。

**3.文件(Files)**

文件在 Linux 系统中是一种基本的持久化存储机制，可用于**进程间通信**。多个进程可以通过对同一个文件的读取和写入来共享信息。

**4.信号量(Semaphores)**
信号量是一种在进程间或同一进程的不同线程间提供同步的机制。它是一个计数器，用于控制对共享资源的访问。当计数器值大于0时，表示资源可用；当值为0时，表示资源被占用。进程在访问共享资源前必须减少（wait）信号量，访问后必须增加（post）信号量。

**5.共享内存(Shared Memory)**
在 Linux 中，共享内存是进程间通信（IPC）的一种形式。当多个进程需要访问相同的数据时，使用共享内存是一种高效的方式。它允许两个或多个进程访问同一个物理内存区域，这使得数据传输不需要通过内核空间，从而提高了通信效率。

**6.消息队列 (Message Queues)**

消息队列是一种允许一个或多个进程向其写入消息，并由一个或多个进程读取消息的 IPC 机制。每条消息都由一个消息队列标识符（ID）识别， 且可以携带一个特定的类型。消息队列允许不同进程非阻塞地发送和接收记录或数据块，这些记录可以是不同类型和大小的。

**7.套接字 (Sockets)**

套接字是一种在不同进程间进行数据交换的通信机制。在 Linux 中，套接字可以用于同一台机器上的进程间通信（IPC）或不同机器上的网络通信。套接字支持多种通信协议，最常见的是TCP（可靠的、连接导向的协议）和UDP（无连接的、不可靠的协议）。

## 七、常用指令

```

uname -m /*查看系统架构*/
lscpu /*查看更多CPU情况*/

tar -vxf [xxx.tar.gz压缩包]
mv [现在的位置] [将要移动的位置]

pip list /*罗列所有的安装包*/

make clean  /*清理编译*/
make && make install  /*开始编译*/

cp  -r /usr/local/include/vod_sdk/ /usr/include/vod_sdk/  /*复制文件夹到另一个文件夹上*/

sudo apt-get autoremove xxx  /*卸载*/
```

常用指令汇总：[linux 常用命令【编程必备】-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/1561151?spm=a2c6h.12873639.article-detail.33.657a1be0lxiKr4&scm=20140722.ID_community@@article@@1561151._.ID_community@@article@@1561151-OR_rec-V_1-RL_community@@article@@1457993)
