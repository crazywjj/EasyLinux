[TOC]





# CentOS7防火墙firewalld配置

![image-20220519103440147](assets/image-20220519103440147.png)

# 1 介绍

Linux的防火墙体系主要工作在网络层，针对TCP/IP数据包实时过滤和限制，属于典型的包过滤防火墙（或称为网络层防火墙）。Linux系统的防火墙体系基于内核共存：firewalld、iptables、ebtables，默认使用firewalld来管理netfilter子系统。

- netfilter：指的是Linux内核中实现包过滤防火墙的内部结构，不以程序或文件的形式存在，属于“内核态”的防火墙功能体系；
- firewalld：指用来管理Linux防护墙的命令程序，属于“用户态”的防火墙管理体系；



**(1) firewalld概述**

"firewalld"是firewall daemon。它提供了一个动态管理的防火墙，带有一个非常强大的过滤系统，称为 Netfilter，由 Linux 内核提供。firewalld是自CentOS 7以来带有一个动态的、可定制而无需重新启动防火墙守护程序或服务。firewall-cmd就是iptables/nftable的前端。在CentOS 8中，nftables取代iptables成为默认的Linux网络包过滤框架。

firewalld的作用是为包过滤机制提供匹配规则（或称为策略），通过各种不同的规则，告诉netfilter对来自指定源，前往指定目的或具有某些协议特征的数据包采取何种处理方式。为了更加方便地组织和管理防火墙，firewalld提供了支持网络区域所定义的网络链接以及接口安全等级的动态防火墙管理工具。支持IPv4、IPv6防火墙设置以及以太网桥，并且拥有两种配置模式：

- 运行配置
- 永久配置

还支持服务或应用程序直接添加防火墙规则接口。

**(2) firewalld网络区域**

firewalld将所有的网络数据流量划分为多个区域，从而简化防火墙管理。根据数据包的源IP地址或传入网络接口等条件，将数据流量转入相应区域的防火墙规则。

对于进入系统的数据包，首先检查的就是其源地址：

- 若源地址关联到特定的区域，则执行该区域所制定的规则；
- 若源地址未关联到特定的区域，则使用传入网络接口的区域并执行该区域所制定的规则；
- 若网络接口未关联到特定的区域，则使用默认区域并执行该区域所制定的规则；

默认区域不是单独的区域，而是指向系统上定义的某个其他区域。默认情况下，默认区域是public，但是也可以更改默认区域。以上匹配规则，按照先后顺序，第一个匹配的规则胜出。在每个区域中都可以配置其要打开或者关闭的一系列服务或端口，firewalld的每个预定义的区域都设置了默认打开的服务。

**(3) firewalld预定义区域说明**

**过滤规则集合：zone**

一个zone就是一套过滤规则，数据包必须要经过某个zone才能入站或出站。不同zone中规则粒度粗细、安全强度都不尽相同。可以把zone看作是一个个出站或入站必须经过的安检门，有的严格、有的宽松、有的检查细致、有的检查粗略。

firewalld将网卡对应到不同的区域（zone），zone 默认共有9个，block ,dmz ,drop external,home,internal ,public , trusted , work.

不同的区域之间的差异是其对待数据包的默认行为不同，根据区域名字我们可以很直观的知道该区域的特征，在CentOS7系统中，默认区域被设置为public。每个zones都可以按照指定的标准进行配置，以根据你的要求接受或拒绝某些服务或端口，并且它可以与一个或多个网络接口相关联。默认区域为public区域。

firewalld使用zones和services的概念，而 iptables 使用chain和rules。与 iptables 相比，“FirewallD”提供了一种非常灵活的方式来处理防火墙管理。

- Block（阻塞）
  任何对该区域的连接请求都会被以 IPv4 的 icmp-host-prohibited 信息或 IPv6 的 icmp6-adm-prohibited 信息所拒绝。只能从系统内部启动网络连接。
- Dmz（隔离）
  用于你的隔离区内的电脑，此区域内可公开访问，可以有限地进入你的内部网络，仅仅接收经过选择的连接。
- Drop（丢弃）
  对进入该区域的所有数据包丢弃，并且不进行任何回包，区域内主动发起连接的流入回程数据包允许通过，允许进行出方向的网络连接。
- External（外部）
  用于在启用伪装的外部网络上使用，尤其路由器、防火墙认为在这个网络上的其它主机不可信。仅仅接收经过选择的连接。
- Home（家庭）
  默认其他同区域内主机可信，仅仅接收经过选择的连接。同时默认放行 ssh、mdns、ipp-client、amba-client 与 dhcpv6-client 服务产生的连接。
- Internal（内部）
  从描述中可以等同于家庭区域。
- Public（公开）
  公共区域，也是防火墙配置的默认区域，防火墙认为该区域主机不可信。仅仅接收经过选择的连接。同时默认放行 ssh 与 dhcpv6-client 服务产生的连接。
- Trusted（可信）
  可信区域，防火墙放行一切流量。等同于关闭防火墙功能。
- Work（工作）
  工作区域，防火墙认为在这个网络上的其它主机不可信。仅仅接收经过选择的连接。同时默认放行 ssh、ipp-client 与 dhcpv6-client 服务产生的连接。



# 2 安装与管理 FirewallD

[CentOS](https://www.linuxprobe.com/) 7 和 Fedora 20+ 已经包含了 FirewallD，但是默认没有激活。可以像其它的 systemd 单元那样控制它。

在Centos 7系统中，可以使用三种方式配置firewalld防火墙：

- firewalld-config图形化工具；
- firewalld-cmd命令行工具；
- /etc/firewalld/中的配置文件；

一般情况下，不建议直接编辑配置文件；

**安装**

```
yum install firewalld firewall-config
```

**1、 启动服务，并在系统引导时启动该服务：**

```
sudo systemctl start firewalld
sudo systemctl enable firewalld
```

要停止并禁用：

```
sudo systemctl stop firewalld
sudo systemctl disable firewalld
```

**2、 检查防火墙状态。输出应该是 running或者 not running。**

```
sudo firewall-cmd --state
```

**3、 要查看 FirewallD 守护进程的状态：**

```
sudo systemctl status firewalld
```

示例输出：

```
firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled)
   Active: active (running) since Wed 2015-09-02 18:03:22 UTC; 1min 12s ago
 Main PID: 11954 (firewalld)
   CGroup: /system.slice/firewalld.service
   └─11954 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid
```

**4、 重新加载 FirewallD 配置：**

```
sudo firewall-cmd --reload
```





# 3 配置 FirewallD

FirewallD 使用 XML 进行配置。除非是非常特殊的配置，你不必处理它们，而应该使用 firewall-cmd

配置文件位于两个目录中：
/usr/lib/FirewallD下保存默认配置，如默认区域和公用服务。避免修改它们，因为每次 firewall 软件包更新时都会覆盖这些文件。
/etc/firewalld 下保存系统配置文件。 这些文件将覆盖默认配置。

**配置集**

FirewallD 使用两个配置集：“运行时”和“持久”。 在系统重新启动或重新启动 FirewallD 时，不会保留运行时的配置更改，而对持久配置集的更改不会应用于正在运行的系统。

默认情况下，firewall-cmd 命令适用于运行时配置，但使用 --permanent 标志将保存到持久配置中。要添加和激活持久性规则，你可以使用两种方法之一。

**1、 将规则同时添加到持久规则集和运行时规则集中。**

```
sudo firewall-cmd --zone=public --add-service=http --permanent
sudo firewall-cmd --zone=public --add-service=http
```

**2、 将规则添加到持久规则集中并重新加载 FirewallD。**

```
sudo firewall-cmd --zone=public --add-service=http --permanent
sudo firewall-cmd --reload
```

reload 命令会删除所有运行时配置并应用永久配置。因为 firewalld 动态管理规则集，所以它不会破坏现有的连接和会话。



# 4  zone管理

“区域”是针对给定位置或场景（例如家庭、公共、受信任等）可能具有的各种信任级别的预构建规则集。不同的区域允许不同的网络服务和入站流量类型，而拒绝其他任何流量。 首次启用 FirewallD 后，public 将是默认区域。

区域也可以用于不同的网络接口。例如，要分离内部网络和互联网的接口，你可以在 internal 区域上允许 DHCP，但在 external区域仅允许 HTTP 和 SSH。未明确设置为特定区域的任何接口将添加到默认区域。

要找到默认区域： 

```bash
sudo firewall-cmd --get-default-zone
```

要修改默认区域：

```bash
sudo firewall-cmd --set-default-zone=internal
```

要查看所有活动的zone，请运行以下命令：：

```bash
sudo firewall-cmd --get-active-zones
```

要得到特定区域的所有配置：

```bash
sudo firewall-cmd --zone=public --list-all
```

要得到所有区域的配置： 

```bash
sudo firewall-cmd --list-all-zones
```

通过使用选项"--zone”和“--change-interface”的组合，可以轻松更改zone中的接口。例如，要将“ens33”接口分配给“public”区域，请运行以下命令：

```bash
firewall-cmd --zone=public --change-interface=ens33
firewall-cmd --reload
```

要创建新zone，请使用以下命令。例如，要创建一个名为“test”的新区域，并永久生效，请运行：

```
firewall-cmd --permanent --new-zone=test
firewall-cmd --reload
```





# 5 添加和移除服务

FirewallD 可以根据特定网络服务的预定义规则来允许相关流量。你可以创建自己的自定义系统规则，并将它们添加到任何区域。 默认支持的服务的配置文件位于 `/usr/lib /firewalld/services`，用户创建的服务文件在 `/etc/firewalld/services` 中。

Firewalld的服务，你不需要记住任何端口，并且可以一次性允许所有端口。

例如，执行以下命令允许 samba 服务。samba 服务需要启用以下一组端口：“139/tcp 和 445/tcp”以及“137/udp 和 138/udp”。

添加'samba'服务后，所有端口都会同时激活，因为所有端口信息都在samba服务配置中。下面是Firewalld中预定义的samba的服务配置文件：

```xml
[root@localhost ~]# cat /usr/lib/firewalld/services/samba.xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>Samba</short>
  <description>This option allows you to access and participate in Windows file and printer sharing networks. You need the samba package installed for this option to be useful.</description>
  <port protocol="udp" port="137"/>
  <port protocol="udp" port="138"/>
  <port protocol="tcp" port="139"/>
  <port protocol="tcp" port="445"/>
  <module name="nf_conntrack_netbios_ns"/>
</service>

```

添加和移除服务：

```bash
firewall-cmd --zone=public --add-service=samba --permanent
firewall-cmd --zone=public --remove-service=samba --permanent
firewall-cmd --reload
```

要获取有关 samba 服务的更多信息，请运行以下命令：

```bash
[root@localhost ~]# firewall-cmd --info-service=samba
samba
  ports: 137/udp 138/udp 139/tcp 445/tcp
  protocols:
  source-ports:
  modules: netbios-ns
  destination:

```

要查看默认的可用服务：

```bash
sudo firewall-cmd --get-services
```

 要一次添加多个服务，请执行以下命令。例如，要添加 http 和 https 服务，请运行以下命令：

```bash
[root@localhost ~]# firewall-cmd --permanent --zone=public --add-service={http,https}
success
[root@localhost ~]# firewall-cmd --reload
success
```



# 6 开放和关闭端口

打开特定端口允许用户从外部访问系统，这代表了安全风险。因此，仅在必要时为某些服务打开所需的端口。

要获取当前区域中开放的端口列表，请运行以下命令：

```
firewall-cmd --list-ports 
```

获取public区域中开放的端口列表：

```
firewall-cmd --zone=public --list-ports 
```



查看想开的端口是否已开：

```
firewall-cmd --query-port=8080/tcp
```

添加指定需要开放的端口：

```
firewall-cmd --zone=public --add-port=8080/tcp --permanent
```

重载入添加的端口：

```
firewall-cmd --reload
```

查询指定端口是否开启成功：

```
firewall-cmd --query-port=8080/tcp
```

移除指定端口：

```
firewall-cmd --zone=public --remove-port=8080/tcp --permanent
```



# 7 设置端口转发

端口转发是一种将任何传入网络流量从一个端口转发到另一个内部端口或另一台机器上的外部端口的方法。

**注意：端口转发必须开启IP伪装。**使用下面显示的命令为 `public`区域启用伪装。

```bash
firewall-cmd --permanent --zone=public --add-masquerade
firewall-cmd --reload
```

要删除规则，用 --remove替换 --add。比如：

```bash
firewall-cmd --zone=public --remove-masquerade
```

要检查是否为区域启用了 IP 伪装，请运行以下命令：

```bash
[root@localhost ~]# firewall-cmd --zone=public --query-masquerade
yes
```

显示yes，表示已经开启伪装。

要将端口重定向到同一系统上的另一个端口，例如：将80端口的所有数据包重定向到8080端口：

```bash
[root@localhost ~]# firewall-cmd --permanent --zone=public --add-forward-port=port=80:proto=tcp:toport=8080
success
```

如果要将流量转发到另一台服务器，例如：将所有 80 端口的数据包重定向到 IP 为 10.0.0.75 的服务器上的 8080 端口：

```bash
[root@localhost ~]# firewall-cmd --permanent --zone=public --add-forward-port=port=80:proto=tcp:toport=8080:toaddr=10.0.0.75
success
```

例如，要允许来自特定源地址的流量，仅允许从特定子网连接到服务器，请运行以下命令：

```bash
[root@localhost ~]# firewall-cmd --permanent --zone=home --add-source=192.168.1.0/24
success
```

补充：

```
centos7通过firewalld配置网关服务器
假设内网网段为：192.168.1.0/24 
可访问外网的内网服务器的内网IP为：192.168.1.1 
可访问外网的内网服务器的内网网络接口为：eth1

1) 在192.168.1.1服务器上做如下配置：
开启ip_forward转发
# 在/etc/sysctl.conf中添加
net.ipv4.ip_forward=1
# 然后让其生效
sysctl -p

2) 转发内网段的流量
执行如下firewalld命令：
firewall-cmd --add-masquerade --permanent
firewall-cmd --permanent --direct --passthrough ipv4 -t nat -I POSTROUTING -o eth1 -j MASQUERADE -s 192.168.1.0/24
firewall-cmd --reload

3)添加内网服务器网关
在需要访问外网的内网服务器上添加192.168.1.1为网关即可访问外网了：
route add default gw 192.168.1.1 dev eth1    #临时生效
vi /etc/sysconfig/network-scripts/ifcfg-eth1
GATEWAY=192.168.1.1

systemctl restart network

4) 修改内网服务器的/etc/resolv.conf为，可上外网服务器的resolv.conf
```





# 8 用 FirewallD 构建规则集

例如，以下是如何使用 FirewallD 为你的服务器配置基本规则（如果您正在运行 web 服务器）。

**1、将 eth0的默认区域设置为 dmz。 在所提供的默认区域中，dmz（非军事区）是最适合于这个程序的，因为它只允许 SSH 和 ICMP。**

```
sudo firewall-cmd --set-default-zone=dmz
sudo firewall-cmd --zone=dmz --add-interface=eth0
```

**2、把 HTTP 和 HTTPS 添加永久的服务规则到 dmz 区域中：**

```
sudo firewall-cmd --zone=dmz --add-service=http --permanent
sudo firewall-cmd --zone=dmz --add-service=https --permanent
```

**3、 重新加载 FirewallD 让规则立即生效：**

```
sudo firewall-cmd --reload
```

如果你运行 firewall-cmd --zone=dmz --list-all， 会有下面的输出：

```
dmz (default)
  interfaces: eth0
  sources:
  services: http https ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
```

这告诉我们， dmz区域是我们的默认区域，它被用于 eth0 接口中所有网络的源地址和端口。 允许传入 HTTP（端口 80）、HTTPS（端口 443）和 SSH（端口 22）的流量，并且由于没有 IP 版本控制的限制，这些适用于 IPv4 和 IPv6。 不允许IP 伪装以及端口转发。 我们没有 ICMP 块，所以 ICMP 流量是完全允许的。没有丰富Rich规则，允许所有出站流量。





# 9  高级配置

服务和端口适用于基本配置，但对于高级情景可能会限制较多。 丰富Rich规则和直接Direct接口允许你为任何端口、协议、地址和操作向任何区域 添加完全自定义的防火墙规则。



# 10 丰富规则

丰富规则允许使用易于理解的命令创建更复杂的防火墙规则，但丰富的规则很难记住，可以查看手册 `man firewalld.richlanguage`并找到示例。使用 --add-rich-rule、 --list-rich-rules、 --remove-rich-rule。 和 firewall-cmd命令来管理它们。

```
富规则的一般规则结构如下：
rule
  [source]
  [destination]
  service|port|protocol|icmp-block|icmp-type|masquerade|forward-port|source-port
  [log]
  [audit]
  [accept|reject|drop|mark]
```

要允许来自地址 192.168.0.0/24 的访问，请运行以下命令：

```
[root@server1 ~]# firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.0.0/24" accept'
success
```

要允许来自地址 192.168.0.0/24 的连接访问 ssh 服务，请运行以下命令：

```
[root@server1 ~]# firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.0.0/24" service name="ssh" log prefix="ssh" level="info" accept'
success
```

要拒绝来自192.168.10.0/24的流量访问ssh服务，请运行以下命令：

```
[root@server1 ~]# firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.10.0/24" port port=22 protocol=tcp reject'
success
```

这里有一些常见的例子：

允许来自主机 192.168.0.14 的所有 IPv4 流量。

```
sudo firewall-cmd --zone=public --add-rich-rule 'rule family="ipv4" source address=192.168.0.14 accept'
```

拒绝来自主机 192.168.1.10 到 22 端口的 IPv4 的 TCP 流量。

```
sudo firewall-cmd --zone=public --add-rich-rule 'rule family="ipv4" source address="192.168.1.10" port port=22 protocol=tcp reject'
```

允许来自主机 10.1.0.3 到 80 端口的 IPv4 的 TCP 流量，并将流量转发到 6532 端口上。 

```
sudo firewall-cmd --zone=public --add-rich-rule 'rule family=ipv4 source address=10.1.0.3 forward-port port=80 protocol=tcp to-port=6532'
```

将主机 172.31.4.2 上 80 端口的 IPv4 流量转发到 8080 端口（需要在区域上激活 masquerade）。

```
sudo firewall-cmd --zone=public --add-rich-rule 'rule family=ipv4 forward-port port=80 protocol=tcp to-port=8080 to-addr=172.31.4.2'
```

列出你目前的丰富规则：

```
sudo firewall-cmd --list-rich-rules
```



# 11 Firewalld的Direct规则

Direct规则类似于 iptables 命令，对于熟悉 iptables 命令的用户很有用。或者，您可以编辑 `/etc/firewalld/direct.xml`文件中的规则并重新加载防火墙以激活这些规则。Direct规则主要由服务或应用程序用来添加特定的防火墙规则。

以下Direct规则将在服务器上打开端口 8080：

```
[root@server1 ~]# firewall-cmd --permanent --direct --add-rule ipv4 filter INPUT 0 -p tcp --dport 8081 -j ACCEPT
success
[root@server1 ~]# firewall-cmd --reload
success
```

要列出当前区域中的Direct规则，请运行：

```
[root@server1 ~]# firewall-cmd --direct --get-all-rules 
ipv4 filter INPUT 0 -p tcp --dport 8080 -j ACCEPT
ipv4 filter INPUT 0 -p tcp --dport 8081 -j ACCEPT
```

使用下面命令删除Direct规则：

```
[root@server1 ~]# firewall-cmd --direct --get-all-rules 
ipv4 filter INPUT 0 -p tcp --dport 8080 -j ACCEPT
ipv4 filter INPUT 0 -p tcp --dport 8081 -j ACCEPT
[root@server1 ~]# firewall-cmd --permanent --direct --remove-rule ipv4 filter INPUT 0 -p tcp --dport 8080 -j ACCEPT
success
[root@server1 ~]# firewall-cmd --reload
success
```

如何清空一个表的链？下面是语法和实例：

```
firewall-cmd --direct --remove-rules ipv4 [table] [chain]
[root@server1 ~]# firewall-cmd --permanent --direct --remove-rules ipv4 filter INPUT
success
[root@server1 ~]# firewall-cmd --reload
success
[root@server1 ~]# firewall-cmd --direct --get-all-rules
```







