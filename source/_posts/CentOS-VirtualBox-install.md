---
title: 记在CentOS 6.5 下安装VirtualBox搭建XP虚拟机过程
date: 2014-12-31 15:22:36
tags: [linux,virtualBox]
categories: linux
---
### 目标 ###
 10.45.4.30主机，系统为CentOS 6.5，因为IETest无法真实模拟IE6，7，8，9，10，11的环境，需要搭建几台真实环境的虚拟机，进行测试

### 如何用Xmanger远程连接10.45.4.30 ###

CentOS 6.5默认是连接不上10.45.4.30图形化界面的，需要在CentOS 6.5 开启Xmanager远程桌面登录

具体操作见：http://www.linuxidc.com/Linux/2014-03/98548.htm

### 安装VirtualBox ###
下载VirtualBox-4.3.20-96996-Linux_amd64.run
我下载的地址是：http://download.pchome.net/system/sysenhance/down-87894-1.html
通过XBorwer连接10.45.4.30

在终端运行 ：
sh VirtualBox-4.3.20-96996-Linux_amd64.run

**报错： 缺少GCC ，安装GCC**

    yum -y install gcc
    装完后输入 gcc --version进行验证

**报错： yum [Errno 256] No more mirrors to try 解决方法**

    yum clean all
    yum makecache

**报错： Please install the build and header files for your current Linux kernel.**

     yum install kernel-devel
    ln -s /usr/src/kernels/2.6.32-504.3.3.el6.x86_64/ /usr/src/linux

再次安装，终于成功

    [root@vm Downloads]# sh VirtualBox-4.3.20-96996-Linux_amd64.run
    Verifying archive integrity... All good.
    Uncompressing VirtualBox for Linux installation............
    VirtualBox Version 4.3.20 r96996 (2014-11-21T14:12:48Z) installer
    Please install the build and header files for your current Linux kernel.
    The current kernel version is 2.6.32-431.el6.x86_64
    Problems were found which would prevent VirtualBox from installing.
    Please correct these problems and try again.
    [root@vm Downloads]# sh VirtualBox-4.3.20-96996-Linux_amd64.run
    Verifying archive integrity... All good.
    Uncompressing VirtualBox for Linux installation............
    VirtualBox Version 4.3.20 r96996 (2014-11-21T14:12:48Z) installer
    Installing VirtualBox to /opt/VirtualBox
    Python found: python, installing bindings...
    Building the VirtualBox kernel modules

    VirtualBox has been installed successfully.

    You will find useful information about using VirtualBox in the user manual
      /opt/VirtualBox/UserManual.pdf
    and in the user FAQ
      http://www.virtualbox.org/wiki/User_FAQ

    We hope that you enjoy using VirtualBox.

### 运行VirtualBox ###

**报错： 根目录报空间不足**

    请参考上一篇文章处理 “CentOS 6.5 下根目录报空间不足 处理”


**报错：Error:
Failed to create the VirtualBox COM object.
The application will now terminate.
Start tag expected, '<' not found.
Location: '/home/xxxx/.VirtualBox/VirtualBox.xml', line 1 (0), column 1.
/home/vbox/vbox-3.1.0/src/VBox/Main/VirtualBoxImpl.cpp[420] (nsresult VirtualBox::init()).
"xxxx"=你的用户名**

    删除/home/xxxx/.VirtualBox/VirtualBox.xml，然后就可以进入virtualbox，但是发现原来的虚拟机不见了。
    别急，点击菜单中的Machine--> Add 然后找到 xxx.vbox ，打开就可以了

### 安装XP
**问题：　ghostxp.iso上的很多工具在virtualbox下都不可用**

        第一次的尝试失败
        + partitionmagic不能用
        + 快速4分区不能用
        + 安装ghostxp到C盘不能用
        + WindowsXpPE是可用的　　但到里面安装系统还是不行

必须使用xp正常安装版

找了半天，只能通过迅雷下，公司下不了，晚上回家下，地址如下：
迅雷下载地址：thunder://QUFodHRwOi8vc29mdC51c2Fpa2EuY24vJUU2JTkzJThEJUU0JUJEJTlDJUU3JUIzJUJCJUU3JUJCJTlGL3dpbmRvd3MvV2luZG93c1hQL1ZMLUltYWdlL01TRE4vemgtaGFuc193aW5kb3dzX3hwX3Byb2Zlc3Npb25hbF93aXRoX3NlcnZpY2VfcGFja18zX3g4Nl9jZF92bF94MTQtNzQwNzAuaXNvWlo=Windows XP pro

with sp3 VOL 微软原版（简体中文）正版密钥：MRX3F-47B9T-2487J-KWKMF-RPWBY下面，开始体验虚拟机上安装操作系统


### 注意 ###
 virtualbox上面的 Host键 默认是 右Ctrol键

  virtualbox联网有问题，需要在 设置-> 网络 -> 高级 修改一下网    卡模拟 PCnet -> Intel Pro
