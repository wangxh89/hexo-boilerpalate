---
title: 记CentOS 6.5 下根目录报空间不足 处理
date: 2014-12-30 15:04:37
tags: [linux,centos]
categories: linux
---
### 准备知识
#### LVM是逻辑盘卷管理（Logical Volume Manager）的简称
他是磁盘管理的另一种工具，就目前基本上所有操作系统均支持，LVM是建立在硬盘和分区之上的一个逻辑层，来提高磁盘分区管理的灵活性。通过LVM系统管理员可以轻松管理磁盘分区，如：将若干个磁盘分区连接为一个整块的卷组（volume group），形成一个存储池。管理员可以在卷组上随意创建逻辑卷组（logical volumes），并进一步在逻辑卷组上创建文件系统。管理员通过LVM可以方便的调整存储卷组的大小，并且可以对磁盘存储按照组的方式进行命名、管理和分配

#### 物理存储介质（The physical media）
这里指系统的存储设备：硬盘，如：/dev/hda、/dev/sda等等，是存储系统最低层的存储单元。
#### 物理卷（physicalvolume）
物理卷就是指硬盘分区或从逻辑上与磁盘分区具有同样功能的设备(如RAID)，是LVM的基本存储逻辑块，但和基本的物理存储介质（如分区、磁盘等）比较，却包含有与LVM相关的管理参数。
#### 卷组（Volume Group）
LVM卷组类似于非LVM系统中的物理硬盘，其由物理卷组成。可以在卷组上创建一个或多个“LVM分区”（逻辑卷），LVM卷组由一个或多个物理卷组成。
#### 逻辑卷（logicalvolume）
LVM的逻辑卷类似于非LVM系统中的硬盘分区，在逻辑卷之上可以建立文件系统(比如/home或者/usr等)。
#### PE（physical extent）
每一个物理卷被划分为称为PE(Physical Extents)的基本单元，具有唯一编号的PE是可以被LVM寻址的最小单元。PE的大小是可配置的，默认为4MB。
#### LE（logical extent）
逻辑卷也被划分为被称为LE(Logical Extents) 的可被寻址的基本单位。在同一个卷组中，LE的大小和PE是相同的，并且一一对应。

### 环境
```
[root@3.5.5Biz-46 ~]# lsb_release -a
LSB Version:    :base-4.0-amd64:base-4.0-noarch:core-4.0-amd64:core-4.0-noarch:graphics-4.0-amd64:graphics-4.0-noarch:printing-4.0-amd64:printing-4.0-noarch
Distributor ID:    CentOS
Description:    CentOS release 6.5 (Final)
Release:    6.5
Codename:    Final
```

### 查看系统目录的挂载情况
```
[root@vm ~]# df -k
Filesystem                 1K-blocks     Used Available Use% Mounted on
/dev/mapper/vg00-lvroot      2064208  2047816         0 100% /
tmpfs                        4031404      144   4031260   1% /dev/shm
/dev/sda1                    1032088    63476    916184   7% /boot
/dev/mapper/vg00-lvhome      4128448   139256   3779480   4% /home
/dev/mapper/vg00-lvopt       8256952   321380   7516144   5% /opt
/dev/mapper/vg00-lvtmp       8256952   149900   7687624   2% /tmp
/dev/mapper/vg00-lvusr      10321208  3278988   6517932  34% /usr
/dev/mapper/vg00-lvvar       8256952   355328   7482196   5% /var
/dev/mapper/vg00-lvztesoft 206424760 13963380 181975620   8% /ztesoft
/dev/sr0                     4363088  4363088         0 100% /media/CentOS_6.5_Final
gvfs-fuse-daemon             2064208  2047816         0 100% /root/.gvfs
```
### 确认home目录下没有数据后，将home目录卸载
```
[root@vm /]# umount /home
[root@vm /]# df -h
Filesystem                  Size  Used Avail Use% Mounted on
/dev/mapper/vg00-lvroot     2.0G  2.0G     0 100% /
tmpfs                       3.9G  144K  3.9G   1% /dev/shm
/dev/sda1                  1008M   62M  895M   7% /boot
/dev/mapper/vg00-lvopt      7.9G  314M  7.2G   5% /opt
/dev/mapper/vg00-lvtmp      7.9G  147M  7.4G   2% /tmp
/dev/mapper/vg00-lvusr      9.9G  3.2G  6.3G  34% /usr
/dev/mapper/vg00-lvvar      7.9G  347M  7.2G   5% /var
/dev/mapper/vg00-lvztesoft  197G   14G  174G   8% /ztesoft
/dev/sr0                    4.2G  4.2G     0 100% /media/CentOS_6.5_Final
gvfs-fuse-daemon            2.0G  2.0G     0 100% /root/.gvfs
```

### 删除lv_home
```
[root@vm /]# lvremove /dev/mapper/vg00-lvhome
Do you really want to remove active logical volume lvhome? [y/n]: y
```
### 将所有空闲空间增加给逻辑卷lv_root
```
[root@vm VirtualBox]# lvextend -L +4G /dev/mapper/vg00-lvroot
  Extending logical volume lvroot to 6.00 GiB
  /etc/lvm/backup/vg00.tmp: write error failed: No space left on device
  Backup of volume group vg00 metadata failed.
  Logical volume lvroot successfully resized
此时查看文件系统的根目录空间并没有发生变化
```
### 调整文件系统空间
```
[root@vm VirtualBox]# resize2fs -p /dev/mapper/vg00-lvroot
resize2fs 1.41.12 (17-May-2010)
Filesystem at /dev/mapper/vg00-lvroot is mounted on /; on-line resizing required
old desc_blocks = 1, new_desc_blocks = 1
Performing an on-line resize of /dev/mapper/vg00-lvroot to 1572864 (4k) blocks.
The filesystem on /dev/mapper/vg00-lvroot is now 1572864 blocks long.
```

### 修改操作系统挂载目录的配置文件/etc/fstab，注释掉home挂载
```
[root@vm VirtualBox]# more /etc/fstab

#
# /etc/fstab
# Created by anaconda on Fri Dec  5 02:40:08 2014
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/vg00-lvroot /                       ext4    defaults        1 1
UUID=307dc23f-1bbc-4c1d-ac15-a26ecf9f5fe3 /boot                   ext4    defaults        1 2
#/dev/mapper/vg00-lvhome /home                   ext4    defaults        1 2
/dev/mapper/vg00-lvopt  /opt                    ext4    defaults        1 2
/dev/mapper/vg00-lvtmp  /tmp                    ext4    defaults        1 2
/dev/mapper/vg00-lvusr  /usr                    ext4    defaults        1 2
/dev/mapper/vg00-lvvar  /var                    ext4    defaults        1 2
/dev/mapper/vg00-lvswap swap                    swap    defaults        0 0
tmpfs                   /dev/shm                tmpfs   defaults        0 0
devpts                  /dev/pts                devpts  gid=5,mode=620  0 0
sysfs                   /sys                    sysfs   defaults        0 0
proc                    /proc                   proc    defaults        0 0
/dev/mapper/vg00-lvztesoft /ztesoft ext4 rw 0 0
```
### 有空间VG 扩充的方法
以上方法对于没有空闲VG ，如果有空闲VG直接操作
```
[root@vm home]# vgs
VG #PV #LV #SN Attr VSize VFree
vg00 1 7 0 wz—n- 298.97g 250.97g
root@vm home]# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/vg00-lvroot
                       99G  1.7G   92G   2% /
tmpfs                 7.8G   68K  7.8G   1% /dev/shm
/dev/sda1            1008M   88M  870M  10% /boot
/dev/mapper/vg00-lvhome
                      4.0G  136M  3.7G   4% /home
/dev/mapper/vg00-lvopt
                      7.9G  146M  7.4G   2% /opt
/dev/mapper/vg00-lvtmp
                      7.9G  147M  7.4G   2% /tmp
/dev/mapper/vg00-lvusr
                      9.9G  9.4G     0 100% /usr

----我直接使用的根目录，所以扩根分区
 [root@vm ~]# lvextend -L 10240M /dev/vg00/lvroot
  Size of logical volume vg00/lvroot changed from 2.00 GiB (64 extents) to 10.00 GiB (320 extents).
  Logical volume lvroot successfully resized
[root@vm ~]# lvextend -L 102400M /dev/vg00/lvroot
  Size of logical volume vg00/lvroot changed from 10.00 GiB (320 extents) to 100.00 GiB (3200 extents).
  Logical volume lvroot successfully resized

----扩展到文件系统
[root@vm ~]# resize2fs /dev/vg00/lvroot
resize2fs 1.41.12 (17-May-2010)
Filesystem at /dev/vg00/lvroot is mounted on /; on-line resizing required
old desc_blocks = 1, new_desc_blocks = 7
Performing an on-line resize of /dev/vg00/lvroot to 26214400 (4k) blocks.
```
