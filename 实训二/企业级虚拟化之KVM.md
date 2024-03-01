[TOC]







# 企业级虚拟化之KVM

# 1.1 KVM简介

Kernel-based Virtual Machine的简称，是一个开源的系统虚拟化模块，自Linux 2.6.20之后集成在Linux的各个主要发行版本中。它使用Linux自身的调度器进行管理，所以相对于Xen，其核心源码很少。KVM目前已成为学术界的主流VMM之一。

KVM的虚拟化需要硬件支持（如Intel VT技术或者AMD V技术)。是基于硬件的完全虚拟化。而Xen早期则是基于软件模拟的Para-Virtualization，新版本则是基于硬件支持的完全虚拟化。但Xen本身有自己的进程调度器，存储管理模块等，所以代码较为庞大。广为流传的商业系统虚拟化软件VMware ESX系列是基于软件模拟的Full-Virtualization。

![1581325529019](assets/1581325529019.png)

因为对进程管理比较麻烦,RedHat发布了一个开源项目libvirt。libvirt有命令行工具也有API，可以通过图形化界面，完成对虚拟机的管理。大多数管理平台通过libvirt来完成对KVM虚拟机的管理；比如Openstack、Cloudstack、OpenNebula等。



## 1.1.1 虚拟化概念

**软件模拟**
优点：能够模拟任何硬件，包括不存在的
缺点：功能非常低效，一般用于研究，生产环境不同。
代表：QEMU

**虚拟化层翻译**
软件全虚拟化----VMware
半虚拟化----改动虚拟机的内核（linux）xen（被淘汰）
硬件支持的全虚拟化----KVM

**容器虚拟化 docker**

**虚拟化分类**
1.硬件虚拟化 硬件虚拟化代表：KVM 
2.软件虚拟化 软件虚拟化代表：Qemu

提示：硬件虚拟化是需要CPU支持，如果CPU不支持将无法创建KVM虚拟机。Qemu和KVM的最大区别就是，如果一台物理机内存直接4G，创建一个vm虚拟机分配内存分4G，在创建一个还可以分4G。支持超配，但是qemu不支持





## 1.1.2 KVM、QEMU、libvirt以及virt-manager等组件的关系

**QEMU：**

QUME提供了一个开源的全虚拟化的解决方案，实际就是一台硬件模拟器，可以模拟许多硬件，包括X86架构处理器、AMD64架构处理器等。QEMU的优点是因为是纯软件模拟，所以可以在支持的平台模拟支持的设备。缺点是因为纯软件模拟，所以非常慢。

KVM只是一个内核模块，只能提供CPU和内存；所以还需要QEMU模拟IO设备；如磁盘、网卡等。

**KVM：**


KVM是Linux内核中的可加载的木块，是一个基于内核的虚拟机。在硬件支持虚拟化(intel VT,AMD-V)的X86平台上实现了全虚拟化功能，由于用户不能直接操作内核，因此需要一个用户空间工具进行操作，通过与QEMU的结合，就可以通过QEMU去操作KVM虚拟机。

**libvirt：重点**


libvirt是为了更方便地管理平台虚拟化技术而设计的开放源代码的应用程序接口、守护进程和管理工具，它不仅提供了对虚拟化客户机的管理，也提供了对虚拟化网络和存储的管理。尽管libvirt项目最初是为Xen设计的一套API，但是目前对KVM等其他Hypervisor的支持也非常的好。libvirt支持多种虚拟化方案，既支持包括KVM、QEMU、Xen、VMware、VirtualBox等在内的平台虚拟化方案，又支持OpenVZ、LXC等Linux容器虚拟化系统，还支持用户态Linux（UML）的虚拟化。


libvirt其实质就是对针对不同的hypervisor的命令进行了一个封装，libvirt针对不同的开发语言提供了api接口，如python、c等；libvirtd是linux的一个守护进程，使用libvirt必须先启动这个守护进程。

Libvirt是一套开源的虚拟化管理工具，主要由3部分组成。

- 一套API的lib库，支持主流的编程语言，包括C、Python、Ruby等
- Libvirt服务
- 命令行工具virsh

**==Libvirt可以实现对虚拟机的管理，比如虚拟机的创建、启动、关闭、暂停、恢复、迁移、销毁，以及对虚拟网卡、硬盘、CPU、内存等多种设备的热添加。==**

**其他组件：**


因为libvirt是目前使用最为广泛的对KVM虚拟机进行管理的工具和应用程序接口（API），而且一些常用的虚拟机管理工具（如virsh、virt-install、virt-manager等）和云计算框架平台（如OpenStack、OpenNebula、Eucalyptus等）都在底层使用libvirt的应用程序接口。


libvirt作为中间适配层，让底层Hypervisor对上层用户空间的管理工具是可以做到完全透明的，因为libvirt屏蔽了底层各种Hypervisor的细节，为上层管理工具提供了一个统一的、较稳定的接口（API）。

![kvm-api](assets/kvm-api.jpg)



**总结：**


KVM是内核的模块；QEMU是提供虚拟化的组件，用户操作KVM模块；libvirt提供一整套的API，用于管理KVM虚拟机，其他图形化界面（virt-manager等）可以通过libvirt管理kvm虚拟机。



# 1.2 KVM安装

## 1.2.1 环境介绍

| 主机名      | 角色    | 外网IP     | 内网IP       | 备注 |
| ----------- | ------- | ---------- | ------------ | ---- |
| CentOS7-200 | 宿主机  | 10.0.0.200 | 172.16.1.200 |      |
| localhost   | kvm虚机 | 10.0.0.100 |              |      |

**kvm相关安装包及其作用**

```bash
qemu-kvm          主要的KVM程序包
python-virtinst   创建虚拟机所需要的命令行工具和程序库
virt-manager      GUI虚拟机管理工具
virt-top          虚拟机统计命令
virt-viewer       GUI连接程序，连接到已配置好的虚拟机
libvirt           C语言工具包，提供libvirt服务
libvirt-client    虚拟客户机提供的C语言工具包
virt-install      基于libvirt服务的虚拟机创建命令
bridge-utils      创建和管理桥接设备的工具
```



## 1.2.2 硬件环境

![1581326252534](assets/1581326252534.png)

![1581326260666](assets/1581326260666.png)



**虚拟化Intel使用的是intel VT-X AMD使用的是AMD-V**



## 1.2.3 系统环境

**检测系统版本及内核**

```shell
[root@ CentOS7-200 ~]# cat /etc/redhat-release
CentOS Linux release 7.3.1611 (Core)

[root@ CentOS7-200 ~]# uname -r
3.10.0-514.el7.x86_64

[root@ CentOS7-200 ~]# getenforce
Disabled

[root@ CentOS7-200 ~]# systemctl stop firewalld.service
```

**检查CPU是否支持虚拟化**
vmx       ##（for Intel CPU）
svm       ##（for AMD CPU）

KVM其实已经在Centos7内置到系统内核，无需安装。

```shell
[root@ CentOS7-200 ~]# egrep -o '(vmx|svm)' /proc/cpuinfo
```

**检查CPU是否开启虚拟化**

在linux平台下，我们可以通过dmesg |grep kvm命令来查看。
如果CPU没有开启虚拟化的话，显示如下：

![1581326922771](assets/1581326922771.png)

**安装kvm用户态模块**

```shell
[root@ CentOS7-200 ~]# yum install qemu-kvm qemu-kvm-tools libvirt -y
```

- libvirt 用来管理kvm

- kvm属于内核态，不需要安装。但是需要一些类似于依赖的
- qemu

**启动libvirt**

```shell
systemctl start libvirtd.service && systemctl enable libvirtd.service
```

libvirt数据目录结构

```bash
$ tree /var/lib/libvirt
.
├── boot
├── dnsmasq
│   ├── default.addnhosts
│   ├── default.conf
│   ├── default.hostsfile
│   └── virbr0.status
├── filesystems
├── images
├── lxc
├── network
├── qemu
│   ├── channel
│   │   └── target
│   ├── dump
│   ├── nvram
│   ├── ram
│   │   └── libvirt
│   │       └── qemu
│   ├── save
│   └── snapshot
└── swtpm
```

**查看网络信息**

启动之后我们可以使用ifconfig或ip命令进行查看，libvirtd已经为我们安装了一个桥接网卡

```shell
[root@ CentOS7-200 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:68:e8:4e brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.200/24 brd 10.0.0.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe68:e84e/64 scope link
       valid_lft forever preferred_lft forever
3: ens34: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:68:e8:58 brd ff:ff:ff:ff:ff:ff
    inet 172.16.1.200/16 brd 172.16.255.255 scope global ens34
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe68:e858/64 scope link
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 52:54:00:56:64:a4 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN group default qlen 1000
    link/ether 52:54:00:56:64:a4 brd ff:ff:ff:ff:ff:ff
```



# 1.3 KVM宿主机网络配置

kvm客户机网络连接有两种方式：

用户网络（User Networking）：NAT方式，让虚拟机访问主机、互联网或本地网络上的资源的简单方法，但是不能从网络或其他的客户机访问客户机，性能上也需要大的调整。

虚拟网桥（Virtual Bridge）：Bridge方式，这种方式要比用户网络复杂一些，但是设置好客户机与互联网，客户机与主机之间的通信都很容易。

==**(建议先配置宿主机桥接网络→创建虚机)**==

![1581328566929](assets/1581328566929.png)

在该模式下，宿主机会虚拟出来一张虚拟网卡作为宿主机本身的通信网卡，而宿主机的物理网卡则成为桥设备（交换机），所以虚拟机相当于在宿主机所在局域网内的一个单独的主机，他的行为和宿主机是同等地位的，没有依存关系。

安装好虚拟化组件(RHEL6.0之后，系统自带的均是KVM，已经没有XEN虚拟化的支持了），会自动生成一个virbr0这样的桥接设备。

```shell
[root@ CentOS7-200 ~]# brctl  show
bridge name	bridge id		STP enabled	interfaces
docker0		8000.0242e20b14dc	no
virbr0		8000.5254005f3794	yes		virbr0-nic   #生成这个
```

Bridge设备其实就是网桥设备，也就相当于想在的二层交换机，用于连接同一网段内的所有机器，所以我们的目的就是将网络设备ens33配置成br0，此时br0就成为了所谓的交换机设备，我们物理机的ens33也是连接在上面的。

可以查看帮助

```bash
virsh help iface-bridge
```

通过virsh的命令可以直接添加网桥； 相比于手动配置，用virsh配置桥接网卡可以不用重启网络。 

```bash
[root@ CentOS7-200 ~]# virsh iface-bridge ens33 br0
Created bridge br0 with attached device ens33
Bridge interface br0 started

```

然后；查看网络信息

```bash
[root@ CentOS7-200 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br0 state UP group default qlen 1000
    link/ether 00:0c:29:68:e8:4e brd ff:ff:ff:ff:ff:ff
3: ens34: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:68:e8:58 brd ff:ff:ff:ff:ff:ff
    inet 172.16.1.200/16 brd 172.16.255.255 scope global ens34
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe68:e858/64 scope link
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 52:54:00:56:64:a4 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN group default qlen 1000
    link/ether 52:54:00:56:64:a4 brd ff:ff:ff:ff:ff:ff
6: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:0c:29:68:e8:4e brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.200/24 brd 10.0.0.255 scope global br0
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe68:e84e/64 scope link
       valid_lft forever preferred_lft forever
       
[root@ CentOS7-200 ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens33
DEVICE=ens33
ONBOOT=yes
BRIDGE="br0"

[root@ CentOS7-200 ~]# cat /etc/sysconfig/network-scripts/ifcfg-br0
DEVICE="br0"
ONBOOT="yes"
TYPE="Bridge"
BOOTPROTO="none"
IPADDR="10.0.0.200"
NETMASK="255.255.255.0"
GATEWAY="10.0.0.254"
STP="on"
DELAY="0"
```

**==以下为手动配置方式：==**

1、查看物理机网卡设备

```bash
[root@CentOS7-200 ~]# ifconfig virbr0
virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
        ether 52:54:00:86:9b:5b  txqueuelen 1000  (Ethernet)
        RX packets 41  bytes 1468 (1.4 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

2、配置桥接设备br0

```shell
[root@ CentOS7-200 ~]# yum -y install bridge-utils
```

(1) 手动添加临时生效

```shell
[root@ CentOS7-200 ~]# brctl  addbr br0

[root@ CentOS7-200 ~]# brctl  show
bridge name	bridge id		STP enabled	interfaces
br0		8000.000000000000	no
docker0		8000.0242e20b14dc	no
virbr0		8000.5254005f3794	yes		virbr0-nic

[root@ CentOS7-200 ~]#  brctl  addif br0 ens33
执行此步后,会导致xshell与宿主机断开连接,以下操作在宿主机完成.
删除ens33上面的ip地址,将br0上面添加上固定ip地址:

[root@ CentOS7-200 ~]#  ip addr del dev ens33 10.0.0.200/24  //删除ens33上的IP地址
[root@ CentOS7-200 ~]# ifconfig  br0 10.0.0.200/24 up  //配置br0的IP地址并启动设备
[root@ CentOS7-200 ~]#  route add default gw 10.0.0.254 //重新加入默认网关
连接xshell查看是否生效

[root@ CentOS7-200 ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.0.254      0.0.0.0         UG    0      0        0 br0
10.0.0.0        0.0.0.0         255.255.255.0   U     0      0        0 br0
172.16.1.0      0.0.0.0         255.255.255.0   U     100    0        0 ens37
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0

[root@CentOS7-200 ~]# ifconfig br0
br0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.0.200  netmask 255.255.255.0  broadcast 10.0.0.255
        inet6 fe80::20c:29ff:fe0e:41a  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:0e:04:1a  txqueuelen 1000  (Ethernet)
        RX packets 44264  bytes 33757615 (32.1 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 31974  bytes 49194899 (46.9 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        
        
[root@ CentOS7-200 ~]# ping www.baidu.com
PING www.a.shifen.com (61.135.169.121) 56(84) bytes of data.
64 bytes from 61.135.169.121 (61.135.169.121): icmp_seq=1 ttl=128 time=4.95 ms
64 bytes from 61.135.169.121 (61.135.169.121): icmp_seq=2 ttl=128 time=4.19 ms
64 bytes from 61.135.169.121 (61.135.169.121): icmp_seq=3 ttl=128 time=6.30 ms
^C
--- www.a.shifen.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 4.194/5.151/6.301/0.872 ms
```

此时宿主机的ip:10.0.0.200已经绑定到br0网卡；但是服务器重启后就不能生效。

**(2) 通过配置文件配置桥接设备永久生效**

为KVM宿主机创建虚拟网卡，并将物理网卡作为桥设备

```shell
[root@ CentOS7-200 ~]# cp /etc/sysconfig/network-scripts/ifcfg-ens33 .

[root@ CentOS7-200 opt]# vim /etc/sysconfig/network-scripts/ifcfg-ens33
DEVICE=ens33
TYPE=Ethernet
ONBOOT=yes
BRIDGE=br0
NM_CONTROLLED=no

[root@ CentOS7-200 opt]# vim /etc/sysconfig/network-scripts/ifcfg-br0
DEVICE=br0
TYPE=Bridge
ONBOOT=yes
BOOTPROTO=static
IPADDR=10.0.0.200
NETMASK=255.255.255.0
GATEWAY=10.0.0.254
NM_CONTROLLED=no

[root@ CentOS7-200 opt]# systemctl restart network.service
```

**(3) 通过VNC连接KVM虚机修改网卡配置**

```shell
[root@ CentOS7-200 ~]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 -     c73                            running
 
[root@ CentOS7-200 ~]# systemctl stop firewalld.service


# 编辑KVM虚出来的机器
[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-ens33
DEVICE=ens33
TYPE=Ethernet
BOOTPROTO=static
ONBOOT=yes
IPADDR=10.0.0.100
PREFIX=24
GATEWAY=10.0.0.254
DNS1=223.5.5.5     此处配置后则不需要手动添加/etc/resolv.conf
DNS2=1.1.1.1

[root@localhost ~]# vim /etc/resolv.conf   #必须有否则xshell连不上
nameserver 223.5.5.5
[root@localhost ~]# ifup ens33
```

注意：此时宿主机还需要通过图形化工具设置网卡为桥接方式，否则无法ping通网关和外网。





# 1.4 创建虚拟机

创建虚拟机步骤

1.准备虚拟机硬盘

2.需要系统iso镜像

3.需要安装一个vnc的客户端来连接



## 1.4.1 查看磁盘空间

最好是20G以上

```shell
[root@ CentOS7-200 ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cl-root   28G   14G   15G  50% /
devtmpfs             2.0G     0  2.0G   0% /dev
tmpfs                2.0G     0  2.0G   0% /dev/shm
tmpfs                2.0G  8.7M  2.0G   1% /run
tmpfs                2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/sda1           1014M  121M  894M  12% /boot
tmpfs                394M     0  394M   0% /run/user/0
```



## 1.4.2 生成镜像 

使用dd命令，复制系统的镜像并输出到iso文件。需要挂载上光盘即可。

```shell
dd if=/dev/cdrom of=/var/lib/libvirt/images/CentOS-7-x86_64-DVD-1908.iso
```

- if=文件名：输入文件名，缺省为标准输入。即指定源文件。< if=input file >
- of=文件名：输出文件名，缺省为标准输出。即指定目的文件。< of=output file >

如果镜像大于4G；或者不想用本机版本的系统；建议使用xftp上传新的镜像。



## 1.4.3 创建磁盘 

提示： qemu-img软件包是我们安装qemu-kvm-tools 依赖给安装上的

```shell
[root@ CentOS7-200 opt]# qemu-img create -f qcow2 /opt/c73.qcow2 6G
```

- -f 制定虚拟机格式

- /opt/ 存放路径
- 6G 代表磁盘大小

**磁盘格式介绍**

raw----裸磁盘不支持快照

qcow2----支持快照。Openstack使用的方式推荐使用这个。注意：关闭虚拟机后操作。

**区别：**

全镜像格式（典型代表raw），特点：设置多大就是多大，写入速度快，方便转换其他格式，性能最优，但是占用空间大。

稀疏格式（典型代表qcow2）,其特点：支持压缩、快照、镜像，更小的存储空间（即用多少占多少）

qcow2 数据的基本组成单元是cluster 

raw性能比qcow2快

raw创建多大磁盘，就占用多大空间直接分配，qcow2动态的用多大占用多大空间。



## 1.4.4 安装虚拟机

安装方式有如下：

<img src="assets/image-20200918165656462.png" alt="image-20200918165656462" style="zoom:67%;" />

```shell
[root@ CentOS7-200 opt]# yum install -y virt-install

[root@ CentOS7-200 opt]# virt-install \
--name=k8s-01 \
--virt-type=kvm \
--vcpus=4 \
--ram=16384 \
--cdrom=/home/iso/CentOS-7-x86_64-Minimal-1810.iso \
--network bridge=br0 \
--disk path=/data0/k8s-01.qcow2,size=1,format=qcow2 \
--graphics vnc,listen=0.0.0.0 \
--noautoconsole \
--os-type=linux \
--os-variant=rhel7

#默认连接端口是从5900开始的
    [root@ CentOS7-200 opt]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 3     c73                            running
 
[root@ CentOS7-200 opt]# netstat -lntup|grep 5900
tcp        0      0 0.0.0.0:5900            0.0.0.0:*               LISTEN      8440/qemu-kvm
```

**virt-install常用参数**

```shell
[root@Dell ~]# virt-install --help
usage: virt-install --name NAME --memory MB STORAGE INSTALL [options]

从指定安装源创建新虚拟机。

optional arguments:
  -h, --help            show this help message and exit
  --version             show program's version number and exit
  --connect URI         通过 libvirt URI 连接到虚拟机管理程序

通用选项:
  -n NAME, --name NAME  客户机实例名称
  --memory MEMORY       Configure guest memory allocation. Ex:
                        --memory 1024 (in MiB)
                        --memory 512,maxmemory=1024
                        --memory 512,maxmemory=1024,hotplugmemorymax=2048,hotplugmemoryslots=2
  --vcpus VCPUS         配置客户机虚拟 CPU(vcpu) 数量。例如：
                        --vcpus 5
                        --vcpus 5,maxcpus=10,cpuset=1-4,6,8
                        --vcpus sockets=2,cores=4,threads=2
  --cpu CPU             CPU model and features. Ex:
                        --cpu coreduo,+x2apic
                        --cpu host-passthrough
                        --cpu host
  --metadata METADATA   配置客户机元数据。例如：
                        --metadata name=foo,title="My pretty title",uuid=...
                        --metadata description="My nice long description"

安装方法选项:
  --cdrom CDROM         光驱安装介质
  -l LOCATION, --location LOCATION
                        安装源 (例如：nfs:host:/path, http://host/path,
                        ftp://host/path)
  --pxe                 使用 PXE 协议从网络引导
  --import              在已有的磁盘镜像中构建客户机
  --livecd              将光驱介质视为 Live CD
  -x EXTRA_ARGS, --extra-args EXTRA_ARGS
                        将附加参数添加到由 --location
                        引导的内核中
  --initrd-inject INITRD_INJECT
                        添加指定文件到由 --location 指定的 initrd
                        根中
  --os-variant DISTRO_VARIANT
                        在客户机上安装的操作系统，例如：'fedor
                        a18'、'rhel6'、'winxp' 等。
  --boot BOOT           配置客户机引导设置。例如：
                        --boot hd,cdrom,menu=on
                        --boot init=/sbin/init (针对容器)
  --idmap IDMAP         为 LXC 容器启用用户名称空间。例如：
                        --idmap uid_start=0,uid_target=1000,uid_count=10

设备选项:
  --disk DISK           指定存储的各种选项。例如：
                        --disk size=10 (在默认位置创建 10GiB 镜像)
                        --disk /my/existing/disk,cache=none
                        --disk device=cdrom,bus=scsi
                        --disk=?
  -w NETWORK, --network NETWORK
                        配置客户机网络接口。例如：
                        --network bridge=mybr0
                        --network network=my_libvirt_virtual_net
                        --network network=mynet,model=virtio,mac=00:11...
                        --network none
                        --network help
  --graphics GRAPHICS   配置客户机显示设置。例如：
                        --graphics vnc
                        --graphics spice,port=5901,tlsport=5902
                        --graphics none
                        --graphics vnc,password=foobar,port=5910,keymap=ja
  --controller CONTROLLER
                        配置客户机控制器设备。例如：
                        --controller type=usb,model=ich9-ehci1
  --input INPUT         配置客户机输入设备。例如：
                        --input tablet
                        --input keyboard,bus=usb
  --serial SERIAL       配置客户机串口设备
  --parallel PARALLEL   配置客户机并口设备
  --channel CHANNEL     配置客户机通信通道
  --console CONSOLE     配置文本控制台连接主机与客户机
  --hostdev HOSTDEV     配置物理 USB/PCI 等主机设备与客户机共享
  --filesystem FILESYSTEM
                        传递主机目录到客户机。例如：
                        --filesystem /my/source/dir,/dir/in/guest
                        --filesystem template_name,/,type=template
  --sound [SOUND]       配置客户机声音设备仿真
  --watchdog WATCHDOG   配置客户机 watchdog 设备
  --video VIDEO         配置客户机视频硬件。
  --smartcard SMARTCARD
                        配置客户机智能卡设备。例如：
                        --smartcard mode=passthrough
  --redirdev REDIRDEV   配置客户机重定向设备。例如：
                        --redirdev usb,type=tcp,server=192.168.1.1:4000
  --memballoon MEMBALLOON
                        配置客户机 memballoon 设备。例如：
                        --memballoon model=virtio
  --tpm TPM             配置客户机 TPM 设备。例如：
                        --tpm /dev/tpm
  --rng RNG             Configure a guest RNG device. Ex:
                        --rng /dev/urandom
  --panic PANIC         配置客户机 panic 设备。例如：
                        --panic default
  --memdev MEMDEV       Configure a guest memory device. Ex:
                        --memdev dimm,target_size=1024

客户机配置选项:
  --security SECURITY   设置域安全驱动配置。
  --cputune CPUTUNE     Tune CPU parameters for the domain process.
  --numatune NUMATUNE   为域进程调整 NUMA 策略。
  --memtune MEMTUNE     为域进程调整内存策略。
  --blkiotune BLKIOTUNE
                        为域进程调整 blkio 策略。
  --memorybacking MEMORYBACKING
                        为域进程设置内存后备策略。例如：
                        --memorybacking hugepages=on
  --features FEATURES   设置域 <features> XML。例如：
                        --features acpi=off
                        --features apic=on,eoi=on
  --clock CLOCK         设置域 <clock> XML。例如：
                        --clock offset=localtime,rtc_tickpolicy=catchup
  --pm PM               配置 VM 电源管理功能
  --events EVENTS       配置 VM 生命周期管理策略
  --resource RESOURCE   配置 VM 资源分区(cgroups)
  --sysinfo SYSINFO     Configure SMBIOS System Information. Ex:
                        --sysinfo emulate
                        --sysinfo host
                        --sysinfo bios_vendor=Vendor_Inc.,bios_version=1.2.3-abc,...
                        --sysinfo system_manufacturer=System_Corp.,system_product=Computer,...
                        --sysinfo baseBoard_manufacturer=Baseboard_Corp.,baseBoard_product=Motherboard,...
  --qemu-commandline QEMU_COMMANDLINE
                        Pass arguments directly to the qemu emulator. Ex:
                        --qemu-commandline='-display gtk,gl=on'
                        --qemu-commandline env=DISPLAY=:0.1

虚拟化平台选项:
  -v, --hvm             这个客户机应该是一个全虚拟化客户机
  -p, --paravirt        这个客户机应该是一个半虚拟化客户机
  --container           这个客户机应该是一个容器客户机
  --virt-type HV_TYPE   要使用的管理程序名称 (kvm, qemu, xen, ...)
  --arch ARCH           模拟 CPU 架构
  --machine MACHINE     机器类型为仿真类型

其它选项:
  --autostart           主机启动时自动启动域。
  --transient           Create a transient domain.
  --wait WAIT           请等待数分钟以便完成安装。
  --noautoconsole       不要自动尝试连接到客户端控制台
  --noreboot            安装完成后不启动客户机。
  --print-xml [XMLONLY]
                        打印生成的 XML 域，而不是创建客户机。
  --dry-run             运行安装程序，但不创建设备或定义客户
                        机。
  --check CHECK         启用或禁用验证检查。例如：
                        --check path_in_use=off
                        --check all=off
  -q, --quiet           抑制非错误输出
  -d, --debug           输入故障排除信息
```

**virt-install详细参数**

安装方法：指定安装方法、GuestOS类型等；  

```bash
-c CDROM, --cdrom=CDROM：光盘安装介质； 
-l LOCATION, --location=LOCATION：安装源URL，支持FTP、HTTP及NFS等，如ftp://172.16.0.1/pub；
--pxe：基于PXE完成安装； 
--livecd: 把光盘当作LiveCD； 
--os-type=DISTRO_TYPE：操作系统类型，如linux、unix或windows等； 
--os-variant=DISTRO_VARIANT：某类型操作系统的变体，如rhel5、fedora8、debian10等； 
-x EXTRA, --extra-args=EXTRA：根据--location指定的方式安装GuestOS时，用于传递给内核的额外选项，例如指定kickstart文件的位置，--extra-args "ks=http://172.16.0.1/class.cfg" 
--boot=BOOTOPTS：指定安装过程完成后的配置选项，如指定引导设备次序、使用指定的而非安装的kernel/initrd来引导系统启动等 ； 例如： 
 --boot  cdrom,hd,network：指定引导次序； 
 --boot kernel=KERNEL,initrd=INITRD,kernel_args=”console=/dev/ttyS0”：指定启动系统的内核及initrd文件； 
```

存储配置：指定存储类型、位置及属性等； 

```bash
--disk=DISKOPTS：指定存储设备及其属性；格式为--disk /some/storage/path,opt1=val1，opt2=val2等；常用的选项有： 
 device：设备类型，如cdrom、disk或floppy等，默认为disk； 
 bus：磁盘总结类型，其值可以为ide、scsi、usb、virtio或xen； 
 perms：访问权限，如rw、ro或sh（共享的可读写），默认为rw； 
 size：新建磁盘映像的大小，单位为GB； 
 cache：缓存模型，其值有none、writethrouth（缓存读）及writeback（缓存读写）； 
 format：磁盘映像格式，如raw、qcow2、vmdk等； 
 sparse：磁盘映像使用稀疏格式，即不立即分配指定大小的空间； 
--nodisks：不使用本地磁盘，在LiveCD模式中常用；
```

网络配置：指定网络接口的网络类型及接口属性如MAC地址、驱动模式等；  

```bash
-w NETWORK, --network=NETWORK,opt1=val1,opt2=val2：将虚拟机连入宿主机的网络中，其中NETWORK可以为： 
 bridge=BRIDGE：连接至名为“BRIDEG”的桥设备； 
 network=NAME：连接至名为“NAME”的网络；　
```

其它常用的选项还有：  

```bash
model：GuestOS中看到的网络设备型号，如e1000、rtl8139或virtio等； 
 mac：固定的MAC地址；省略此选项时将使用随机地址，但无论何种方式，对于KVM来说，其前三段必须为52:54:00； 
--nonetworks：虚拟机不使用网络功能；
```

图形配置：定义虚拟机显示功能相关的配置，如VNC相关配置；  

```bash
--graphics TYPE,opt1=val1,opt2=val2：指定图形显示相关的配置，此选项不会配置任何显示硬件（如显卡），而是仅指定虚拟机启动后对其进行访问的接口； 
 TYPE：指定显示类型，可以为vnc、sdl、spice或none等，默认为vnc； 
 port：TYPE为vnc或spice时其监听的端口； 
 listen：TYPE为vnc或spice时所监听的IP地址，默认为127.0.0.1，可以通过修改/etc/libvirt/qemu.conf定义新的默认值； 
 password：TYPE为vnc或spice时，为远程访问监听的服务进指定认证密码； 
--noautoconsole：禁止自动连接至虚拟机的控制台；
```

设备选项：指定文本控制台、声音设备、串行接口、并行接口、显示接口等；  

```bash
--serial=CHAROPTS：附加一个串行设备至当前虚拟机，根据设备类型的不同，可以使用不同的选项，格式为“--serial type,opt1=val1,opt2=val2,...”，例如： 
 --serial pty：创建伪终端； 
 --serial dev,path=HOSTPATH：附加主机设备至此虚拟机； 
--video=VIDEO：指定显卡设备模型，可用取值为cirrus、vga、qxl或vmvga；
```

虚拟化平台：虚拟化模型（hvm或paravirt）、模拟的CPU平台类型、模拟的主机类型、hypervisor类型（如kvm、xen或qemu等）以及当前虚拟机的UUID等；  

```bash
-v, --hvm：当物理机同时支持完全虚拟化和半虚拟化时，指定使用完全虚拟化； 
 -p, --paravirt：指定使用半虚拟化； 
 --virt-type：使用的hypervisor，如kvm、qemu、xen等；所有可用值可以使用’virsh capabilities’命令获取；
```

其它：  

```bash
--autostart：指定虚拟机是否在物理启动后自动启动； 
--print-xml：如果虚拟机不需要安装过程(--import、--boot)，则显示生成的XML而不是创建此虚拟机；默认情况下，此选项仍会创建磁盘映像； 
--force：禁止命令进入交互式模式，如果有需要回答yes或no选项，则自动回答为yes； 
--dry-run：执行创建虚拟机的整个过程，但不真正创建虚拟机、改变主机上的设备配置信息及将其创建的需求通知给libvirt； 
-d, --debug：显示debug信息； 
```

**示例：**

1、创建一个名为rhel5的虚拟机，其hypervisor为KVM，内存大小为512MB，磁盘为8G的映像文件/var/lib/libvirt/p_w_picpaths/rhel5.8.img，通过boot.iso光盘镜像来引导启动安装过程。  

```bash
virt-install \ 
--connect qemu:///system \ 
--virt-type kvm \ 
--name rhel5 \ 
--ram 512 \ 
--disk path=/var/lib/libvirt/p_w_picpaths/rhel5.img,size=8 \ 
--graphics vnc \ 
--cdrom /tmp/boot.iso \ 
--os-variant rhel5
```

2、创建一个名为rhel6的虚拟机，其有两个虚拟CPU，安装方法为FTP，并指定了ks文件的位置，磁盘映像文件为稀疏格式，连接至物理主机上的名为brnet0的桥接网络：  

```bash
virt-install \  
--connect qemu:///system \  
--virt-type kvm \  
--name rhel6 \  
--ram 1024 \  
--vcpus 2 \  
--network bridge=brnet0 \  
--disk path=/VMs/p_w_picpaths/rhel6.img,size=120,sparse \  
--location ftp://192.168.10.7/rhel6/dvd \  
--extra_args “ks=http://192.168.10.7/rhel6.cfg” \  
--os-variant rhel6 \  
--force  
```

3、创建一个名为rhel5.8的虚拟机，磁盘映像文件为稀疏模式的格式为qcow2且总线类型为virtio，安装过程不启动图形界面（--nographics），但会启动一个串行终端将安装过程以字符形式显示在当前文本模式下，虚拟机显卡类型为cirrus：  

```bash
virt-install \ 
--connect qemu:///system \ 
--virt-type kvm \  
--name rhel5.8 \  
--vcpus 2,maxvcpus=4 \ 
--ram 512 \  
--disk path=/VMs/p_w_picpaths/rhel5.8.img,size=120,format=qcow2,bus=virtio,sparse \  
--network bridge=brnet0,model=virtio 
--nographics \ 
--location ftp://172.16.0.1/pub \  
--extra-args "ks=http://172.16.0.1/class.cfg  console=ttyS0  serial" \ 
--os-variant rhel5 \ 
--force　
```

下面的示例则利用已经存在的磁盘映像文件（已经有安装好的系统）创建一个名为rhel5.8的虚拟机：  

```bash
virt-install \ 
--name rhel5.8 
--ram 512 
--disk /VMs/rhel5.8.img 
--import 
```

4、通过网络引导安装系统

```bash
virt-install -n "centos6.5" \
--vcpus 2 \
-r 512 \
-l http://192.168.8.42/cobbler/ks_mirror/centos6.5-x86_64/ \
--disk path=/p_w_picpaths/vm3/centos65.qcow2,bus=virtio,size=120,sparse \
--network bridge=br0,model=virtio \
--force
```

5、通过网络引导且通过kickstart文件自动化安装系统

```bash
virt-install -n "test" \
--vcpus 4 \
-r 8192 \
-l http://10.159.237.1/cobbler/ks_mirror/CentOS-7.4-x86_64/ \
--extra-args "ks=http://10.159.237.1/cobbler/CentOS-7.4.ks" \
--disk path=/data0/test.qcow2,bus=virtio,size=120,sparse \
--network bridge=br0,model=virtio \
--force
或者
virt-install \
--name=test \
--virt-type=kvm \
--vcpus=4 \
--ram=8192 \
--pxe \
--network bridge=br0 \
--disk path=/data0/test.qcow2,size=40,format=qcow2 \
--graphics vnc,listen=0.0.0.0 \
--noautoconsole \
--force \
--os-type=linux \
--os-variant=rhel7
```

 每个虚拟机创建后，其配置信息保存在/etc/libvirt/qemu目录中，文件名与虚拟机相同，格式为XML  



## 1.4.5 VNC连接虚拟机

![1581328308002](assets/1581328308002.png)

安装系统

![1581328380058](assets/1581328380058.png)

注意：如果查看5900端口开启，但是VNC无法连接KVM虚拟机时，看下防火墙是否开启。创建的虚机用VNC连接时从默认端口5900开始,即虚机一:10.0.0.200:5900  虚机二:10.0.0200:5901

虚拟机安装完成后是关闭了，我们需要启动

```shell
[root@ CentOS7-200 opt]# virsh list --all
Id    Name                           State
- 	  c73                            shut off

[root@ CentOS7-200 opt]# virsh start c73
```

> c73 是虚拟机的名字，是我们创建的时候定义的



**常用的virsh管理命令**

```shell
列出所有的虚拟机	virsh list --all
显示虚拟机信息		virsh dominfo c73
列出ID为6的虚拟机名 virsh domname 6
显示虚拟机内存和cpu的使用情况 	virt-top
关闭虚拟机 		   virsh shutdown c73 
强制关闭虚拟机 	  virsh destroy c73 
启动虚拟机 			  virsh start c73 
设置虚拟机随系统自启 	  virsh autostart c73 
关闭虚拟机随系统自启	  virsh autostart --disable c73 
删除虚拟机			  virsh undefine c73 
通过控制窗口登录虚拟机   virsh console c73 
挂起$hostname虚拟机 	  virsh suspend c73 
恢复挂起的虚拟机		virsh resume c73 
查看网卡配置信息		virsh domiflist c73 
查看该虚拟机的磁盘位置	  virsh domblklist  c73 
查看KVM虚拟机当前配置    virsh dumpxml c73 
```



# 1.5 KVM图形管理工具（virt-manager）

virt-manager是用于管理KVM虚拟环境的主要工具，virt-manager默认设置下需要使用root用户才能够使用该工具。当你想在KVM hypervisor服务器上托管虚拟机，由最终用户而非root用户访问这些虚拟机时并不总是很便利。

virt-manager可以设置本机，同样也可以连接远程宿主机来管理。

通过virt-manager图形化配置虚拟机，并安装操作系统，是类似于VMware workstation安装虚拟机的过程。很明显，这些操作包括以下几个过程：

1. 配置虚拟机
2. 为虚拟机安装操作系统
3. 其他管理虚拟机的操作

这个过程比较适合自定义安装，比如配置多个NIC，DISK等，或者安装Windows等操作系统。

virt-manager的功能就是用来图形化管理虚拟机，其中包括虚拟机的创建，管理等操作。 

## 1.5.1 安装virt-manager

如果宿主机系统已经是图形化界面；则无需执行以下操作。

直接执行`yum install virt-manager -y`

**1、查看sshd是否开启X11转发**

```shell
[root@ CentOS7-200 ~]# grep X11Forwarding /etc/ssh/sshd_config --colour
X11Forwarding yes

# X11Forwarding no
```

**2、安装virt-manager** 

libvirt是管理虚拟机的API库，不仅支持KVM虚拟机，也可以管理Xen等方案下的虚拟机。

```shell
[root@ CentOS7-200 ~]# yum install virt-manager libvirt libvirt-client virt-viewer qemu-kvm mesa-libglapi -y
[root@ CentOS7-200 ~]# systemctl restart libvirtd.service && systemctl enable libvirtd.service
```



**以下废弃：**

因为我的主机是服务器，没有图形化界面，想要用virt-manager图形化安装虚拟机，还需要安装X-window。

```shell
[root@ CentOS7-200 ~]# yum install libXdmcp libXmu libxkbfile xkeyboard-config xorg-x11-xauth xorg-x11-xkb-utils xorg-x11-font-utils.x86_64 xorg-x11-server-utils.x86_64 xorg-x11-utils.x86_64 xorg-x11-xauth.x86_64 xorg-x11-xinit.x86_64 xorg-x11-drv-ati-firmware -y
```



## 1.5.2 配置xshell

**安装好Xming后**，打开xshell连接会话，[文件-->属性-->隧道-->开启转发X11连接到（x DISPLAY:localhost:0.0）]在连接属性的tunneing中，勾选 Forwarding X11 connection to选项，可以正常打开virt-manager的图形界面。

![1581329484340](assets/1581329484340.png)



## 1.5.3 启动virt-manager

断开xshell会话，重新连接，windows打开Xming；KVM宿主机输入命令：virt-manager，就可以自动弹出kvm管理软件

```shell
[root@ CentOS7-200 ~]# virt-manager

# 1、无法显示报错：
(virt-manager:4957): Gtk-WARNING **: 17:53:43.003: cannot open display: localhost:10.0
# 解决措施：
打开Xming的安装目录，找到文件“X0.hosts”（刚安装的Xming改文件名一般为X0，也可能X1，此处不讨论），以文本形式打开这个文件，将远程机器的IP地址添加到文件中的内容如下：
localhost
10.0.0.200

# 2、virt-manager启动报缺少驱动
libGL error: unable to load driver: swrast_dri.so
libGL error: failed to load driver: swrast

# 终极方案：
yum install -y mesa*
```

然后启动 virt-manager：

![1581329518427](assets/1581329518427.png)

出现乱码，请安装以下包：

```shell
yum install dejavu-sans-mono-fonts -y
```

配置虚机网络：

![1589627442304](assets/1589627442304.png)

![1589627485320](assets/1589627485320.png)



## 1.5.4 管理多台kvm宿主机

![image-20200921103030880](assets/image-20200921103030880.png)



如果出现提示you need to install openssh-askpass or similar to connect to this host
解决办法两种
（1）安装openssh-askpass
（2）或者输入命令：virt-manager --no-fork 命令行输入密码

<img src="assets/image-20200921181117955.png" alt="image-20200921181117955" style="zoom:67%;" />

# 1.6 虚机磁盘迁移

关闭虚机，将磁盘文件转移至新位置

```bash
mv /var/lib/libvirt/images/xxx.qcow2 /data0/vmhosts/win7.img
```

编辑对应虚机配置文件，修改source file项为磁盘文件转移后的位置

![image-20210319141456806](assets/image-20210319141456806.png)

重新引用虚机配置文件，即可开启虚机

```bash
virsh define /etc/libvirt/qemu/xxx.xml
```





# 1.7 虚机克隆

kvm虚拟机的克隆分为两种情况，第一种kvm宿主机上对虚拟机直接克隆（也可称为链接克隆）；第二种通过复制配置文件与磁盘文件的虚拟机复制克隆(也可称为完全克隆，适用于异机的静态迁移)。

**克隆前提条件：**

- 删除ifcfg-eth0网卡中的UUID
- 清空/etc/udev/rules.d/70-persistent-net.rules



方法一：kvm宿主机上对虚拟机直接克隆（需要在关机或暂停的状态下操作）

查看所有的虚拟机、以及需要克隆的虚拟机的硬盘文件的位置

```bash
[root@localhost ~]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 4     win7                           running
 -     centos7.4mini                  shut off
 -     centos7.4mini-clone-40         shut off

[root@localhost ~]# virsh edit centos7.4mini
<source file='/data0/vmhosts/centos7.qcow2'/>


[root@localhost ~]# virt-clone -o centos7.4mini -n centos7.4mini-clone-41 -f /data0/vmhosts/centos7.4mini-clone-41.qcow2

#清理虚机
停止主机：virsh destroy linux65
删除主机定义：virsh undefine linux65
删除KVM虚拟机文件： rm -f /home/vps/linux65.img
```



方法二：复制配置文件与磁盘文件进行克隆（可以不用关闭源虚拟机）

```bash
[root@localhost vmhosts]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 4     win7                           running
 -     centos7.4mini                  shut off
 -     centos7.4mini-clone-40         shut off
 -     centos7.4mini-clone-41         shut off

[root@localhost vmhosts]# virsh dumpxml centos7.4mini >/etc/libvirt/qemu/centos7.4mini-clone-42.xml
[root@localhost vmhosts]# cp /data0/vmhosts/centos7.qcow2 /data0/vmhosts/centos7.4mini-clone-42.qcow2
#直接编辑修改配置文件centos7.4mini-clone-42.xml，修改name,uuid,disk文件位置,mac地址，vnc端口

```



# 1.8 虚机开机自启

在kvm图形化管理工具里面可以设置，让kvm虚拟机随着宿主虚拟机一起启动。

必须在关机状态下做。

![20d184db3a2c4a8a8a72a7bb7992ec26](assets/20d184db3a2c4a8a8a72a7bb7992ec26.jpg)

![de9ea692692f49d1a9a2a96fbfe52017](assets/de9ea692692f49d1a9a2a96fbfe52017.jpg)

设置好以后会像Windows一样创建一个快捷方式

```
[root@CentOS2 ~]# cd /etc/libvirt/qemu/autostart/
[root@CentOS2 autostart]# ls
centos7.0.xml
```

如果取消开机自动启动那个勾选，这个xml就不会被创建。

以上是使用图形界面方式设置kvm虚拟机开机自动启动。下面演示命令行方式

```bash
[root@CentOS2 autostart]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 -     centos7.0                      shut off
 -     winxp                          shut off

[root@CentOS2 autostart]# virsh autostart --disable centos7.0
Domain centos7.0 unmarked as autostarted

[root@CentOS2 autostart]# ls /etc/libvirt/qemu/autostart/
[root@CentOS2 autostart]# virsh autostart  centos7.0
Domain centos7.0 marked as autostarted

[root@CentOS2 autostart]# ls /etc/libvirt/qemu/autostart/
centos7.0.xml
```















