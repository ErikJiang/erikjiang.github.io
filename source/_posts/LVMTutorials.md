---
title: "LVM 逻辑卷管理器的使用"
date: 2020-05-15 19:37:03
categories: "技术" 
tags: 
  - kube
  - lvm
type: "tags"
---

逻辑卷管理器 LVM (Logical Volume Manager), 又称为逻辑卷宗管理器、逻辑磁盘管理器，是Linux核心所提供的逻辑卷管理功能。它在硬盘分区之上，又创建一个逻辑层，以便系统管理硬盘分割系统。

最早由IBM开发，在AIX系统上实现，OS/2 操作系统与 HP-UX也支持这个功能。在1998年，Heinz Mauelshagen 根据在 HP-UX 上的逻辑卷管理器，编写出第一个 Linux 版本的逻辑卷管理器。

<!--more-->

### LVM 的基本概念术语：

* PV(Physical volume): 物理卷, PV处于LVM系统最低层, 它可以是整个硬盘, 或者与磁盘分区具有相同功能的设备(如RAID)
* VG(Volume Group): 卷组, 创建在PV之上, 由一个或多个PV组成,可以在VG上创建一个或多个“LVM分区”(逻辑卷),功能类似非LVM系统的物理硬盘
* LV(Logical Volume): 逻辑卷, 从VG中分割出的一块空间, 创建之后其大小可以伸缩，在LV上可以创建文件系统(如/var,/home)
* PE(Physical extent): 物理块, 每一个PV被划分为基本单元(也被称为PE), 具有唯一编号的PE是可以被LVM寻址的最小存储单元, 默认为4MB

```
                         Disk A                                                                  Disk B
+-------------------------------------------------------------+               +---------------------------------------+
|                                                             |               |                                       |
|                                                             |               |                                       |
|       PV1                   PV2                  PV3        |               |       PV1                  PV2        |
+------------------+  +------------------+ +------------------+               +------------------+ +------------------+
|------------------|  |------------------| |------------------|               |------------------| |------------------|
|| PE || PE || PE ||  || PE || PE || PE || || PE || PE || PE ||               || PE || PE || PE || || PE || PE || PE ||
|------------------|  |------------------| |------------------|               |------------------| |------------------|
+-------------------------------------------------------------+               +---------------------------------------+
         |                      |                   |                                   |                    |
         |                      |                   |                             +-----+                    |
         |                      |                   |                             |                          |
         |   +-----------+      |                   |                       +-----v-----+                    |
         +-->+    VG1    +<-----+                   +---------------------->+    VG2    +<-------------------+
             +-----+-----+                                                  +-----+-----+
                   |                                                              |
                   |                                                +-------------+------------+
                   |                                                |                          |
              +----v-----+                                     +----v-----+               +----v-----+
              |   LV1    |                                     |   LV2    |               |   LV3    |
              | (/home)  |                                     | (/usr)   |               | (/data)  |
              +----------+                                     +----------+               +----------+
```
### 安装 LVM 工具

CentOS 中通过 yum 包管理工具安装:
``` sh
yum install lvm2* -y
```

### 物理磁盘分区

首先对新挂载磁盘进行分区, 这里将整个磁盘作为一个分区配置:
```sh
[root@node01 ~]# fdisk /dev/vdd # 对 /dev/vdd 进行分区
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0xfb5eb0bf.

Command (m for help): n # 新建分区
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p # 主分区
Partition number (1-4, default 1): 1 # 分区号
First sector (2048-209715199, default 2048): # 默认: 首扇区号
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-209715199, default 209715199): # 默认: 尾扇区号
Using default value 209715199
Partition 1 of type Linux and of size 100 GiB is set
# 下面分区类型设置很重要!
Command (m for help): t # 改变类型(分区系统ID)
Selected partition 1
Hex code (type L to list all codes): 8e # 8e 表示 LVM 的分区编号
Changed type of partition 'Linux' to 'Linux LVM'
Command (m for help): w # 保存配置
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```
注意, 如果是要对一个磁盘做多个分区, 则需要根据分区规划的大小, 多次对`Partition number`, `First sector`, `Last sector`进行调整划分;


磁盘分区设置完成, 查看分区表, 会有看到如下`Linux LVM` 分区信息:
```
# fdisk -l
   Device Boot      Start         End      Blocks   Id  System
/dev/vdd1            2048   209715199   104856576   8e  Linux LVM
```

### 准备物理卷(PV)

依赖于磁盘分区`/dev/vdd1` 创建物理卷:
``` sh
[root@node01 ~]# pvcreate /dev/vdd1
  Physical volume "/dev/vdd1" successfully created.
```

展示已创建的物理卷:
``` sh
[root@node01 ~]# pvdisplay 
  "/dev/vdd1" is a new physical volume of "<100.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/vdd1
  VG Name
  PV Size               <100.00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               n0tSls-bw30-tNpv-ogjB-ILOW-vvxq-xgp2Oq
```

删除物理卷命令:
``` sh
# pvremove ${pv_name}
```

### 准备卷组(VG)

创建新卷组, 使用物理卷`/dev/vdd1`创建卷组`volume-group1`:
``` sh
[root@node01 ~]# vgcreate volume-group1 /dev/vdd1
  Volume group "volume-group1" successfully created
```

展示已创建的卷组:
``` sh
[root@node01 ~]# vgdisplay
  --- Volume group ---
  VG Name               volume-group1
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <100.00 GiB
  PE Size               4.00 MiB
  Total PE              25599
  Alloc PE / Size       0 / 0
  Free  PE / Size       25599 / <100.00 GiB
  VG UUID               F3T1Ix-5t4S-tcfI-OZEE-wrAE-wyNc-qDIeNX
```

删除卷组命令:
``` sh
# vgremove ${vg_name}
```

> 若卷组容量已满, 需要扩展卷组大小:
``` sh
[root@node01 ~]# vgextend volume-group1 /dev/vdc1
  Volume group "volume-group1" successfully extended
[root@node01 ~]# vgdisplay
  --- Volume group ---
  VG Name               volume-group1
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               199.99 GiB
  PE Size               4.00 MiB
  Total PE              51198
  Alloc PE / Size       25575 / 99.90 GiB
  Free  PE / Size       25623 / <100.09 GiB
  VG UUID               F3T1Ix-5t4S-tcfI-OZEE-wrAE-wyNc-qDIeNX
```

### 准备逻辑卷(LV)

使用卷组`volume-group1`来创建逻辑卷(名字:lv1, 大小:99.9G):
``` sh
[root@node01 ~]# lvcreate -L 99.9G -n lv1 volume-group1
  Rounding up size to full physical extent 99.90 GiB
  Logical volume "lv1" created.
```

展示已创建的逻辑卷:
``` sh
[root@node01 ~]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/volume-group1/lv1
  LV Name                lv1
  VG Name                volume-group1
  LV UUID                iRlrj0-PvcQ-wKFk-IR1M-LysP-q2CK-Yy7xUi
  LV Write Access        read/write
  LV Creation host, time node01, 2020-05-14 15:37:17 +0800
  LV Status              available
  # open                 0
  LV Size                99.90 GiB
  Current LE             25575
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           252:0
```

将创建好的逻辑卷格式化为 ext4 分区:
``` sh
[root@node01 ~]# mkfs.ext4 /dev/volume-group1/lv1
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
6553600 inodes, 26188800 blocks
1309440 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2174746624
800 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	4096000, 7962624, 11239424, 20480000, 23887872

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```
注: `mkfs.ext4` 格式化逻辑卷为不可逆操作, 若逻辑卷中有重要数据, 则请备份后再格式化, 否则无法复原;


挂载逻辑卷, 比如挂载到`/data`:
``` sh
[root@node01 ~]# mount /dev/volume-group1/lv1 /data/

# 查看挂载情况
[root@node01 ~]# df -h -t ext4
Filesystem                      Size  Used Avail Use% Mounted on
/dev/mapper/volume--group1-lv1   99G   61M   94G   1% /data
```

开机时自动挂载逻辑卷, 需要在 `/etc/fstab` 文件中配置:
```
#
# /etc/fstab
# Created by anaconda on Mon Aug 13 11:29:03 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=db6c825d-c763-42ae-8ccb-e8f5eaead61a /                       ext4    defaults        1 1
LABEL=YUNIFYSWAP none    swap    sw      0 0
/dev/volume-group1/lv2       /var/lib/docker         ext4            defaults        1 1
/dev/volume-group1/lv1       /data                   ext4            defaults        1 1
```
这里将逻辑卷`lv1` 与 `lv2`, 分别与 `/data` 和 `/var/lib/docker` 目录进行关联挂载;


检测 `/etc/fstab` 挂载配置是否可用:
```
# mount -a
```
注: 在重启系统前, 一定要做 `mount -a` 检测, 否则可能会有系统无法正常启动的风险!!!

---


### 关于磁盘数据的备份与还原

1. 数据备份前, 先对 k8s 节点进行安全移除, 以便当前 pods 可以转移到其他节点主机:
```
# kubectl drain <node name> --ignore-daemonsets
```

2. 若涉及到 docker 数据备份, 先停用 docker 服务:
```
# systemctl stop docker
# systemctl stop kubelet
```

3. 考虑到磁盘数据备份还原迁移耗时, 可使用 [screen](http://www.gnu.org/software/screen/) 建立会话, 防止数据迁移任务中断:
```
# 备份 /data/ 数据
$ screen -S backup-data
$ cp /data/* /backup/ -avr
$ < ctrl + a + d >

# 还原 /data/ 数据
$ screen -S restore-data
$ cp /backup/* /data/ -avr
$ < ctrl + a + d >

```

4. 数据还原后, 启用 docker 服务:
```
# systemctl start docker
# systemctl start kubelet
```

5. 还原集群节点
```
# kubectl uncordon <node name>
```
    
> 若通过 uncordon 还原后, node 状态为 NotReady
> 可查看 kubelet log:
```
# journalctl -u kubelet
```
> 若错误信息如下, 表明 swap 被开启, 影响到 kubelet 启动, 需要关闭:
```
failed to run Kubelet: Running with swap on is not supported, please disable swap!
```
>关闭 swap 处理方法:
```
# swapoff -a
```

---

* [LVM简明教程](https://linux.cn/article-3218-1.html)