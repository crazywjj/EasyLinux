







# LVM磁盘逻辑卷管理

# 1 介绍

LVM是 Logical Volume Manager(逻辑卷管理)的简写，它由Heinz Mauelshagen在Linux 2.4内核上实现。LVM将一个或多个硬盘的分区在逻辑上集合，相当于一个大硬盘来使用，当硬盘的空间不够使用的时候，可以继续将其它的硬盘的分区加入其中，这样可以实现磁盘空间的动态管理，相对于普通的磁盘分区有很大的灵活性。

与传统的磁盘与分区相比，LVM为计算机提供了更高层次的磁盘存储。它使系统管理员可以更方便的为应用与用户分配存储空间。在LVM管理下的存储卷可以按需要随时改变大小与移除(可能需对文件系统工具进行升级)。LVM也允许按用户组对存储卷进行管理，允许管理员用更直观的名称(如"sales'、 'development')代替物理磁盘名(如'sda'、'sdb')来标识存储卷。



**工作原理：**

LVM在每个物理卷头部都维护了一个metadata，每个metadata中都包含了整个VG（volume group：卷组）的信息，包括每个VG的布局配置，PV（physical volume：物理卷）的编号，LV（logical volume：逻辑卷）的编号，以及每个PE（physical extends：物理扩展单元）到LE（logical extends：物理扩展单元）的映射关系。同一个VG中的每个PV头部的信息都是相同的，这样有利于故障时进行数据恢复。

LVM对上层文件系统提供LV层，隐藏了操作细节。对文件系统而言，对LV的操作与原先对partition的操作没有差别。当对LV进行写入操作的时候，LVM定位相应的LE，通过PV头部的映射表将数据写入到相应的PE上。LVM实现的关LVM最大的特点就是可以对磁盘进行动态管理。因为逻辑卷的大小是可以动态调整的，而且不会丢失现有的数据。我们如果新增加了硬盘，其也不会改变现有上层的逻辑卷。键在于PE和LE之间建立映射关系，不同的映射规则决定了不同的LVM存储模型。LVM支持多个PV 的stripe和mirror。

LVM最大的特点就是可以对磁盘进行动态管理，因为逻辑卷的大小是可以动态调整的，而且不会丢失现有的数据，如果我们增加了硬盘也不会改变现有的上层逻辑卷。

**LVM的优缺点：**

优点：

1. 文件系统可以跨多个磁盘，因此文件系统大小不会受物理磁盘的限制。

2. 可以在系统运行的状态下动态的扩展文件系统的大小。

3. 可以增加新的磁盘到LVM的存储池中。

4. 可以以镜像的方式冗余重要的数据到多个物理磁盘。

5. 可以方便的导出整个卷组到另外一台机器。

缺点：

1. 在从卷组中移除一个磁盘的时候必须使用reducevg命令（这个命令要求root权限，并且不允许在快照卷组中使用）。

2. 当卷组中的一个磁盘损坏时，整个卷组都会受到影响。

3. 因为加入了额外的操作，存贮性能受到影响。


**LVM名词解释**

LVM是在磁盘分区和文件系统之间添加的一个逻辑层，来为文件系统屏蔽下层磁盘分区布局，提供一个抽象的盘卷，在盘卷上建立文件系统。几个LVM术语：

**物理存储介质（The physical media）**：这里指系统的存储设备：硬盘，如：/dev/hda1、/dev/sda等等，是存储系统最低层的存储单元。

**物理卷（physical volume）**：物理卷就是指硬盘分区或从逻辑上与磁盘分区具有同样功能的设备(如RAID)，是LVM的基本存储逻辑块，但和基本的物理存储介质（如分区、磁盘等）比较，却包含有与LVM相关的管理参数。

**卷组（Volume Group）**：LVM卷组类似于非LVM系统中的物理硬盘，其由物理卷组成。可以在卷组上创建一个或多个“LVM分区”（逻辑卷），LVM卷组由一个或多个物理卷组成。

**逻辑卷（logical volume）**：LVM的逻辑卷类似于非LVM系统中的硬盘分区，在逻辑卷之上可以建立文件系统(比如/home或者/usr等)。

**PE（physical extent）**：每一个物理卷被划分为称为PE(Physical Extents)的基本单元，具有唯一编号的PE是可以被LVM寻址的最小单元。PE的大小是可配置的，默认为4MB。

**LE（logical extent）**：逻辑卷也被划分为被称为LE(Logical Extents) 的可被寻址的基本单位。在同一个卷组中，LE的大小和PE是相同的，并且一一对应。

简单来说就是：

PV:是物理的磁盘分区

VG:LVM中的物理的磁盘分区，也就是PV，必须加入VG，可以将VG理解为一个仓库或者是几个大的硬盘。

LV：也就是从VG中划分的逻辑分区

如下图所示PV、VG、LV三者关系：

![201208221004465079](assets/201208221004465079.jpg)





**LVM的写入模式**

**LVM有两种写入模式：线性模式和条带模式**。

- 线性模式即写完一个设备后再写另一个设备
- 条带模式就有点类似于RAID0，即数据是被分散写入到LVM各成员设备上的。
  因为条带模式的数据不具有安全性，且LVM并不强调读写性能，故LVM默认为线性模式，这样即使一个设备坏了，其它设备上的数据还在。



# 2 安装lvm

```
yum install lvm2 -y
```



# 3 创建和管理LVM

lvm常用的命令



| 功能         | PV管理命令 | VG管理命令 | LV管理命令 |
| ------------ | ---------- | ---------- | ---------- |
| scan 扫描    | pvscan     | vgscan     | lvscan     |
| create 创建  | pvcreate   | vgcreate   | lvcreate   |
| display 显示 | pvdisplay  | vgdisplay  | lvdisplay  |
| remove 移除  | pvremove   | vgremove   | lvremove   |
| extend 扩展  |            | vgextend   | lvextend   |
| reduce 减少  |            | vgreduce   | lvreduce   |



## 3.1 磁盘分区

```
[root@localhost ~]# fdisk /dev/vdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x97ad1447.

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-209715199, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-209715199, default 209715199): 100000
Partition 1 of type Linux and of size 47.8 MiB is set

Command (m for help): w
The partition table has been altered!

```



创建pv

```
[root@localhost ~]# pvcreate /dev/vdb{1,2,3}
WARNING: xfs signature detected on /dev/vdb1 at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/vdb1.
  Physical volume "/dev/vdb1" successfully created.
  Physical volume "/dev/vdb2" successfully created.
  Physical volume "/dev/vdb3" successfully created.
[root@localhost ~]# lsblk -af
NAME   FSTYPE      LABEL UUID                                   MOUNTPOINT
sr0
vda
├─vda1 xfs               f0e8b174-489d-4cd4-8945-475ecfc4e3a3   /boot
├─vda2 swap              8d3d48e0-a31b-47dc-9873-86901c715647   [SWAP]
└─vda3 xfs               c4406e41-8ca0-4a2e-92fd-9b6442d5c0eb   /
vdb
├─vdb1 LVM2_member       p9bdfl-Ysf0-eHkN-1fAd-Vko3-1Wl5-catYVx
├─vdb2 LVM2_member       dNLSIk-GUWw-2j3H-maOb-JIcy-JC7I-Ulgb9l
└─vdb3 LVM2_member       4iPV5l-0e4a-yYA5-2HeO-1YY9-NyWM-fBQGkb
[root@localhost ~]# pvs
  PV         VG Fmt  Attr PSize   PFree
  /dev/vdb1     lvm2 ---  <47.83m <47.83m
  /dev/vdb2     lvm2 ---   95.23g  95.23g
  /dev/vdb3     lvm2 ---    4.72g   4.72g

```



创建vg

```
[root@localhost ~]# vgcreate vg_test /dev/vdb{1,2,3}
  Volume group "vg_test" successfully created
[root@localhost ~]# vgs
  VG      #PV #LV #SN Attr   VSize  VFree
  vg_test   3   0   0 wz--n- 99.99g 99.99g

```

激活vg

```
[root@localhost ~]# vgchange -a y vg_test   # 我们上面就是激活状态的，如果我们重启系统，或者vgchange -y n命令关闭了，就需要这个命令启动下
  0 logical volume(s) in volume group "vg_test" now active
```

移除vg

```
[root@localhost ~]# vgchange -a n vg_test  # 要想移除vg，需要先关闭vg才能移除，这里先关闭
  0 logical volume(s) in volume group "vg_test" now active
[root@localhost ~]# vgremove vg_test
  Volume group "vg_test" successfully removed
```

vg添加成员





