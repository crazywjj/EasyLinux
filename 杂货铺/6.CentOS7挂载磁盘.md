[TOC]



# CentOS7 挂载磁盘

> 注意：挂载操作会清空数据，请确认挂载盘无数据或者未使用。

## 1、列出所有磁盘

```bash
[root@scbb01 ~]# lsblk   或者   ll /dev/disk/by-path
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
fd0               2:0    1    4K  0 disk
sda               8:0    0  100G  0 disk
├─sda1            8:1    0  200M  0 part /boot
└─sda2            8:2    0 99.8G  0 part
  ├─centos-root 253:0    0   92G  0 lvm  /
  └─centos-swap 253:1    0  7.8G  0 lvm  [SWAP]
sdb               8:16   0  500G  0 disk
sr0              11:0    1   66M  0 rom
```

注意：一般情况下sda为系统盘已经挂载，新添的磁盘从sdb开始，例如：`sdb、sdc、sdd、sde...`。当无法确认数据盘设备名称，请使用`df -h`命令来确认系统盘的名称，从而排除挂错盘的情况。

## 2、格式化硬盘	

```bash
[root@scbb01 ~]# fdisk /dev/sdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x33bc6b1e.

Command (m for help): n  #新建分区
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p  #创建逻辑分区
Partition number (1-4, default 1): 1  #输入分区号以及指定分区大小,依照提示，回车表示默认。
First sector (2048-1048575999, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-1048575999, default 1048575999):
Using default value 1048575999
Partition 1 of type Linux and of size 500 GiB is set

Command (m for help): p  #检查分区情况（此时还未执行分区操作）

Disk /dev/sdb: 536.9 GB, 536870912000 bytes, 1048576000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x33bc6b1e

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048  1048575999   524286976   83  Linux

Command (m for help): w  #保存退出
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

## 3、创建文件系统

```bash
# 确定一下当前使用的文件系统，以下为xfs
[root@scbb01 ~]# cat /etc/fstab
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=f196d0dd-a595-4177-879c-e5d2cea0a62c /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0

[root@scbb01 ~]# mkfs.xfs -f /dev/sdb1  # mkfs.xfs==mkfs -t xfs  作用等同
meta-data=/dev/sdb1              isize=512    agcount=4, agsize=32767936 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=131071744, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=63999, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

## 4、创建挂载点并挂载硬盘

```bash
[root@scbb01 ~]# mkdir /data
[root@scbb01 ~]# mount -t xfs /dev/sdb1 /data
[root@scbb01 ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   92G  6.6G   86G   8% /
devtmpfs                 3.9G     0  3.9G   0% /dev
tmpfs                    3.9G     0  3.9G   0% /dev/shm
tmpfs                    3.9G  8.6M  3.9G   1% /run
tmpfs                    3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/sda1                197M  121M   77M  62% /boot
tmpfs                    783M     0  783M   0% /run/user/0
/dev/sdb1                500G   33M  500G   1% /data
```

## 5、开机自动挂载

```bash
[root@scbb01 ~]#  echo "/dev/sdb1        /data          xfs    defaults        0 0" >> /etc/fstab
[root@scbb01 ~]# cat /etc/fstab
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=f196d0dd-a595-4177-879c-e5d2cea0a62c /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
/dev/sdb1               /data                   xfs    defaults        0 0
```

注意：此步操作完，最好重启一下，确保磁盘能开机自动挂载。



## 附：磁盘>2T和<2T分区情况

1、当硬盘小于等于2T时，可以用fdisk进行分区

```bash
fdisk /dev/sdb
1、查看帮助。
输入：m
2、新建分区。
输入：n
3、创建逻辑分区
输入：p
4、输入分区号以及指定分区大小
依照提示，回车表示默认。
5、检查分区情况（此时还未执行分区操作）
Command（m for help）：p 
6、保存退出
Command（m for help）：w
```

2、当硬盘大于2T时，用parted命令创建主分区个步骤。

**介绍说明：**

parted的操作都是实时的，也就是说你执行了一个分区的命令，他就实实在在地分区了，

而不是像fdisk那样，需要执行w命令写入所做的修改， 所以进行parted的测试千万注意不能在生产环境中。

传统的MBR(Master Boot Record)分区方式，有一个局限：无法支持超过2TB的硬盘的分区（或单个分区超过2TB），

这个情况在当前这个数据量激增的时候，实在令人难以接受（尤其是企业级的应用，动则数TB的数据量）。

GPT的分区表很好了解决了传统MBR无法逾越2TB的限制。但是在Linux系统中，传统的fdisk命令无法支持gpt分区方式，这时候我们就要用到parted命令，下面介绍parted命令用法。

```bash
root@kvm1:/# parted /dev/sda
GNU Parted 3.2
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) help                                                             
align-check TYPE N                       check partition N for TYPE(min|opt) alignment（检查分区N是否为TYPE（min | opt）对齐）
help [COMMAND]                           print general help, or help on COMMAND（打印一般帮助，或帮助COMMAND）
mklabel,mktable LABEL-TYPE               create a new disklabel (partition table)（创建一个新的disklabel（分区表））
mkpart PART-TYPE [FS-TYPE] START END     make a partition（做一个分区）
name NUMBER NAME                         name partition NUMBER as NAME（将分区名称NUMBER作为NAME）
print [devices|free|list,all|NUMBER]     display the partition table, available devices, free space, all found partitions, or a particular partition（显示分区表，可用设备，可用空间，所有找到的分区或特定分区）
quit                                     exit program（退出程序）
rescue START END                         rescue a lost partition near START and END（在START和END附近找出丢失的分区）
resizepart NUMBER END                    resize partition NUMBER（调整分区NUMBER）
rm NUMBER                                delete partition NUMBER（删除分区NUMBER）
select DEVICE                            choose the device to edit（选择要编辑的设备）
disk_set FLAG STATE                      change the FLAG on selected device（更改所选设备上的FLAG）
disk_toggle [FLAG]                       toggle the state of FLAG on selected device（在所选设备上切换FLAG的状态）
set NUMBER FLAG STATE                    change the FLAG on partition NUMBER（更改分区NUMBER上的FLAG）
toggle [NUMBER [FLAG]]                   toggle the state of FLAG on partition NUMBER（切换分区NUMBER上的FLAG状态）
unit UNIT                                set the default unit to UNIT（将默认单位设置为UNIT）
version                                  display the version number and copyright information of GNU Parted（显示GNU Parted的版本号和版权信息）
```

**用法实例：**

```bash
(parted) /dev/sda print   #打印磁盘当前分区结构
Model: LSI MR9270CV-8i (scsi)
Disk /dev/sda: 8999GB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name  Flags
 1      17.4kB  1049kB  1031kB                     bios_grub
 2      1049kB  538MB   537MB   fat32              boot, esp
 3      538MB   8999GB  8998GB                     lvm

(parted) mklabel gpt #将一个MBR的磁盘格式化为GPT磁盘：

(parted) mklabel msdos  #将一个GPT磁盘格式化为MBR磁盘：

(parted) mkpart primary 0 100M 或者 /dev/sda mkpart primary 0 100M #划分一个起始位置是0，大小为100M的主分区：

(parted) mkpart primary 0 -1 或者 (parted) /dev/sda mkpart primary 0 -1  #将一个磁盘的所有空间都划分成一个分区：

(parted) rm 1   或者 (parted) /dev/sda rm1 #删除一个分区

(parted) p #查看分区

(parted) q #退出

mkfs.xfs /dev/sda1 #格式化已经分好的区，可以用xfs或者ext4，建议xfs

注意：parted命令和fdisk命令不同，fdisk命令是等到你最后执行那个w的时候才生效最终写入到分区表中的，
parted命令是实时的写入到分区表，所以在操作有数据的磁盘的时候需要格外小心，毕竟数据无价的！
```

