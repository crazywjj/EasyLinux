[TOC]



# Linux系统配置双网卡绑定bond0

# 1 bonding简述

双网卡配置设置虚拟为一个网卡实现网卡的冗余，其中一个网卡坏掉后网络通信仍可正常使用，实现网卡层面的负载均衡和高可用性。现在一般的企业都会使用双网卡接入，这样既能添加网络带宽，同时又能做相应的冗余，可以说是好处多多。而一般企业都会使用linux操作系统下自带的网卡绑定模式，当然现在网卡产商也会出一些针对windows操作系统网卡管理软件来做网卡绑定（windows操作系统没有网卡绑定功能 需要第三方支持）。



## 1.1 bonding原理

网卡工作在混杂（promisc）模式，接收到达网卡的所有数据包，tcpdump工作用的也是混杂模式（promisc），将两块网卡的MAC地址修改为相同接收特定MAC的数据帧，然后把相应的数据帧传送给bond驱动程序进行处理。



## 1.2 bonding模式（bonding mode）

网卡绑定mode共有七种(0~6) bond0、bond1、bond2、bond3、bond4、bond5、bond6

- 轮询策略（round robin），mode=0，按照设备顺序依次传输数据包，提供负载均衡和容错能力
- 主备策略（active-backup），mode=1，只有主网卡处于工作状态，备网卡处于备用状态，主网卡坏掉后备网卡开始工作，提供容错能力
- 异或策略（load balancing (xor)），mode=2，根据源MAC地址和目的MAC地址进行异或计算的结果来选择传输设备，提供负载均衡和容错能力
- 广播策略（fault-tolerance (broadcast)），mode=3，将所有数据包传输给所有接口通过全部设备来传输所有数据，一个报文会复制两份通过bond下的两个网卡分别发送出去，提供高容错能力
- 动态链接聚合（lacp），mode=4，按照802.3ad协议的聚合自动配置来共享相同的传输速度，网卡带宽最高可以翻倍，链路聚合控制协议（LACP）自动通知交换机聚合哪些端口，需要交换机支持 802.3ad协议，提供容错能力
- 输出负载均衡模式（transmit load balancing），mode=5，输出负载均衡模式，只有输出实现负载均衡，输入数据时则只选定其中一块网卡接收，需要网卡和驱动支持ethtool命令
- 输入/输出负载均衡模式（adaptive load balancing），mode=6，输入和输出都实现负载均衡，需要网卡和驱动支持ethtool命令



## 1.3 常用的有三种

mode=0：平衡负载模式，有自动备援，但需要”Switch”支援及设定。

mode=1：自动备援模式，其中一条线若断线，其他线路将会自动备援。

mode=6：平衡负载模式，有自动备援，不必”Switch”支援及设定。

需要说明的是如果想做成mode 0的负载均衡,仅仅设置这里options bond0 miimon=100 mode=0是不够的,与网卡相连的交换机必须做特殊配置（这两个端口应该采取聚合方式），因为做bonding的这两块网卡是使用同一个MAC地址.从原理分析一下（bond运行在mode 0下）：

mode 0下bond所绑定的网卡的IP都被修改成相同的mac地址，如果这些网卡都被接在同一个交换机，那么交换机的arp表里这个mac地址对应的端口就有多 个，那么交换机接受到发往这个mac地址的包应该往哪个端口转发呢？正常情况下mac地址是全球唯一的，一个mac地址对应多个端口肯定使交换机迷惑了。所以 mode0下的bond如果连接到交换机，交换机这几个端口应该采取聚合方式（cisco称为 ethernetchannel，foundry称为portgroup），因为交换机做了聚合后，聚合下的几个端口也被捆绑成一个mac地址.我们的解 决办法是，两个网卡接入不同的交换机即可。

mode6模式下无需配置交换机，因为做bonding的这两块网卡是使用不同的MAC地址。



# 2  网口绑定bond

通过网口绑定(bond)技术,可以很容易实现网口冗余，负载均衡，从而达到高可用高可靠的目的。



前提条件：

2个物理网口：eth0,eth1

绑定的虚拟口：bond0

服务器ip地址：10.1.4.3

网关：10.1.4.254



**1、修改配置网卡文件**

创建bond0文件

```shell
vi /etc/sysconfig/network-scripts/ifcfg-bond0

#TYPE=Bond
BOOTPROTO=none
#DEFROUTE=yes
#NAME=bond
DEVICE=bond0
ONBOOT=yes
#BONDING_MASTER=yes
#BONDING_OPTS=mode=802.3ad   # 等同于mode=4
IPADDR=10.1.4.3
NETMASK=255.255.255.0
GATEWAY=10.1.4.254
```

修改eth0、eth1

```shell
vim /etc/sysconfig/network-scripts/ifcfg-eth0

TYPE=Ethernet
DEVICE=eth0
ONBOOT=yes
NAME=bond-slave-eth0
MASTER=bond0
SLAVE=yes

vim /etc/sysconfig/network-scripts/ifcfg-eth1

TYPE=Ethernet
DEVICE=eth1
ONBOOT=yes
NAME=bond-slave-eth1
MASTER=bond0
SLAVE=yes
```

**2、修改modprobe相关设定文件，并加载bonding模块**

注意：如果打开上一步bond0网卡配置的步骤，则不用执行这一步。

```shell
vi /etc/modprobe.d/bonding.conf

#追加
alias bond0 bonding
options bonding mode=4 miimon=200 primary=eth0

# alias bond0 bonding，表示系统在启动时加载bonding模块，对外虚拟网络接口设备为 bond0
# miimon=200，表示系统每100ms监测一次链路连接状态，如果有一条线路不通就转入另一条线
# mode=4，表示绑定模式为4
# primary=eth0，系统首先eth0作为bond0接口与外界信息的传输接口


2.加载模块(重启系统后就不用手动再加载了)
$ modprobe bonding
$ echo "modprobe bonding miimon=100 mode=4" >>/etc/rc.local

3.确认模块是否加载成功：
$ lsmod | grep bonding
bonding 100065 0
```

**3、重启网卡**

```shell
systemctl restart network
# 注意：需要注意一下NetworkManager和network冲突的问题
```



# 附：bond的七种工作模式介绍的详细介绍

第一种模式：mod=0 ，即：(balance-rr) Round-robin policy（平衡抡循环策略）

特点：传输数据包顺序是依次传输（即：第1个包走eth0，下一个包就走eth1….一直循环下去，直到最后一个传输完毕），此模式提供负载平衡和容错能力；但是我们知道如果一个连接或者会话的数据包从不同的接口发出的话，中途再经过不同的链路，在客户端很有可能会出现数据包无序到达的问题，而无序到达的数据包需要重新要求被发送，这样网络的吞吐量就会下降

 

第二种模式：mod=1，即： (active-backup) Active-backup policy（主-备份策略）

特点：只有一个设备处于活动状态，当一个宕掉另一个马上由备份转换为主设备。mac地址是外部可见得，从外面看来，bond的MAC地址是唯一的，以避免switch(交换机)发生混乱。此模式只提供了容错能力；由此可见此算法的优点是可以提供高网络连接的可用性，但是它的资源利用率较低，只有一个接口处于工作状态，在有 N 个网络接口的情况下，资源利用率为1/N

 

第三种模式：mod=2，即：(balance-xor) XOR policy（平衡策略）

特点：基于指定的传输HASH策略传输数据包。缺省的策略是：(源MAC地址 XOR 目标MAC地址) % slave数量。其他的传输策略可以通过xmit_hash_policy选项指定，此模式提供负载平衡和容错能力

 

第四种模式：mod=3，即：broadcast（广播策略）

特点：在每个slave接口上传输每个数据包，此模式提供了容错能力

 

第五种模式：mod=4，即：(802.3ad) IEEE 802.3ad Dynamic link aggregation（IEEE 802.3ad 动态链接聚合）

特点：创建一个聚合组，它们共享同样的速率和双工设定。根据802.3ad规范将多个slave工作在同一个激活的聚合体下。

外出流量的slave选举是基于传输hash策略，该策略可以通过xmit_hash_policy选项从缺省的XOR策略改变到其他策略。需要注意的 是，并不是所有的传输策略都是802.3ad适应的，尤其考虑到在802.3ad标准43.2.4章节提及的包乱序问题。不同的实现可能会有不同的适应 性。

必要条件：

条件1：ethtool支持获取每个slave的速率和双工设定

条件2：switch(交换机)支持IEEE 802.3ad Dynamic link aggregation

条件3：大多数switch(交换机)需要经过特定配置才能支持802.3ad模式

 

第六种模式：mod=5，即：(balance-tlb) Adaptive transmit load balancing（适配器传输负载均衡）

特点：不需要任何特别的switch(交换机)支持的通道bonding。在每个slave上根据当前的负载（根据速度计算）分配外出流量。如果正在接受数据的slave出故障了，另一个slave接管失败的slave的MAC地址。

必要条件：ethtool支持获取每个slave的速率

 

第七种模式：mod=6，即：(balance-alb) Adaptive load balancing（适配器适应性负载均衡）

特点：该模式包含了balance-tlb模式，同时加上针对IPV4流量的接收负载均衡(receive load balance, rlb)，而且不需要任何switch(交换机)的支持。接收负载均衡是通过ARP协商实现的。bonding驱动截获本机发送的ARP应答，并把源硬件地址改写为bond中某个slave的唯一硬件地址，从而使得不同的对端使用不同的硬件地址进行通信。

来自服务器端的接收流量也会被均衡。当本机发送ARP请求时，bonding驱动把对端的IP信息从ARP包中复制并保存下来。当ARP应答从对端到达 时，bonding驱动把它的硬件地址提取出来，并发起一个ARP应答给bond中的某个slave。使用ARP协商进行负载均衡的一个问题是：每次广播 ARP请求时都会使用bond的硬件地址，因此对端学习到这个硬件地址后，接收流量将会全部流向当前的slave。这个问题可以通过给所有的对端发送更新 （ARP应答）来解决，应答中包含他们独一无二的硬件地址，从而导致流量重新分布。当新的slave加入到bond中时，或者某个未激活的slave重新 激活时，接收流量也要重新分布。接收的负载被顺序地分布（round robin）在bond中最高速的slave上

当某个链路被重新接上，或者一个新的slave加入到bond中，接收流量在所有当前激活的slave中全部重新分配，通过使用指定的MAC地址给每个 client发起ARP应答。下面介绍的updelay参数必须被设置为某个大于等于switch(交换机)转发延时的值，从而保证发往对端的ARP应答 不会被switch(交换机)阻截。

必要条件：

条件1：ethtool支持获取每个slave的速率；

条件2：底层驱动支持设置某个设备的硬件地址，从而使得总是有个slave(curr_active_slave)使用bond的硬件地址，同时保证每个bond 中的slave都有一个唯一的硬件地址。如果curr_active_slave出故障，它的硬件地址将会被新选出来的 curr_active_slave接管

其实mod=6与mod=0的区别：mod=6，先把eth0流量占满，再占eth1，….ethX；而mod=0的话，会发现2个口的流量都很稳定，基本一样的带宽。而mod=6，会发现第一个口流量很高，第2个口只占了小部分流量







