# NTP时间同步服务器

# 1.1 NTP 简介

NTP（ Network Time Protocol，网络时间协议）是用来使网络中的各个计算机时间同步的一种协

议。它的用途是把计算机的时钟同步到世界协调时 UTC，其精度在局域网内可达 0.1ms，在互联

网上绝大多数的地方其精度可以达到 1-50ms。

NTP 服务器就是利用 NTP 协议提供时间同步服务的。



## 1.1.1 环境介绍

| 主机名 | 角色      | 系统      | ip地址    | 备注 |
| ------ | --------- | --------- | --------- | ---- |
| c701   | NTP服务端 | CentOS7.7 | 10.0.0.41 |      |
| c702   | NTP客户端 | CentOS7.7 | 10.0.0.42 |      |



## 1.1.2 NTP服务器安装

```shell
yum -y install ntp        
```



## 1.1.3 配置NTP服务

```shell
vim /etc/ntp.conf 

# restrict default kod nomodify notrap nopeer noquery
# nomodify客户端可以同步
restrict default nomodify


# 将默认时间同步源注释改用可用源
# server 0.centos.pool.ntp.org iburst
# server 1.centos.pool.ntp.org iburst
# server 2.centos.pool.ntp.org iburst
# server 3.centos.pool.ntp.org iburst
server ntp1.aliyun.com
```



## 1.1.4 重启ntp并设置开机自启

```shell
systemctl restart ntpd
systemctl enable ntpd
```



# 2.1 客户端同步时间

```shell
[root@ c702 yum.repos.d]# systemctl stop ntpd
[root@ c702 yum.repos.d]# ntpdate 10.0.0.41
 6 Nov 18:36:39 ntpdate[2151]: adjust time server 10.0.0.41 offset -0.019067 sec
```

注意：此处需要等待服务端几分钟；如下报错 `22 Sep 11:54:25 ntpdate[70350]: the NTP socket is in use, exiting`；ntpd服务运行的情况下会导致ntpdate错误。

添加到定时任务

```shell
cat >>/var/spool/cron/root<<EOF
#crond m01
*/5 * * * * /usr/sbin/ntpdate 10.0.0.41 >/dev/null 2>&1
EOF
```





# 3.1 无外网时间同步

**服务端**

1、关闭防火墙和selinux

2、手动修改时间

```
date -s '20200629 08:44:00'
```

设置的时间写到硬件时间中去（也就是CMOS里面的时间）

```
clock -w
hwclock --systohc
```

3、修改ntp配置文件

1）修改/etc/ntp.conf

注释掉原来的`restrict default ignore`这一行，这一行本身是不响应任何的ntp更新请求，其实也就是禁用了本机的ntp server的功能，所以需要注释掉。

2）加入下面3行： 

```bash
restrict 10.10.10.0 mask 255.255.255.0 nomodify notrap  #注释:用于让10.10.10.0/24网段上的机器能和本机做时间同步
server 127.127.1.0  # local clock
fudge 127.127.1.0 stratum 10
```

后两行是让本机的ntpd和本地硬件时间同步。

3）`/etc/init.d/ntpd restart`或者 `service ntpd restart`
4） `chkconfig ntpd on` 设置开机自启动
5）修改iptables配置，将tcp和udp 123端口开放，这是ntp需要的端口，在/etc/services中可以查到这个端口

**客户端**

1、可以使用ntpdate，但是不推荐，因为时间差比较大会发生时间跳跃（建议将ntpdate时间同步开机自动执行一次）。

2、启动ntpd 连接内网服务器做时间同步

查看ntpd 状态

```bash
#/etc/init.d/ntpd status
ntpd is stopped
```

修改ntp的配置文件

```bash
vim  /etc/ntp.conf
server 2.centos.pool.ntp.org iburst
server 10.200.63.134  #指定内网时间同步ip
```

启动ntpd 服务

```bash
/etc/init.d/ntpd start
```

设置开机自启动

```bash
#chkconfig --list|grep ntpd
ntpd            0:off   1:off   2:off   3:off   4:off   5:off   6:off
#chkconfig ntpd on
#chkconfig --list|grep ntpd
ntpd            0:off   1:off   2:on    3:on    4:on    5:on    6:off
```

查看是否连通指定ip

```bash
#ntpq -p
```





# 4.1 ntpd、ntpdate的区别

ntpd与ntpdate在更新时间时有什么区别。ntpd不仅仅是时间同步服务器，它还可以做客户端与标准时间服务器进行同步时间，而且是平滑同步，并非ntpdate立即同步，在生产环境中慎用ntpdate，也正如此两者不可同时运行。

时钟的跃变，对于某些程序会导致很严重的问题。许多应用程序依赖连续的时钟，这是一项常见的假定，即，取得的时间是线性的，一些操作，例如数据库事务，通常会地依赖这样的事实：时间不会往回跳跃。不幸的是，ntpdate调整时间的方式就是我们所说的”跃变“：在获得一个时间之后，ntpdate使用settimeofday(2)设置系统时间，这有几个非常明显的问题：

第一，这样做不安全。ntpdate的设置依赖于ntp服务器的安全性，攻击者可以利用一些软件设计上的缺陷，拿下ntp服务器并令与其同步的服务器执行某些消耗性的任务。由于ntpdate采用的方式是跳变，跟随它的服务器无法知道是否发生了异常（时间不一样的时候，唯一的办法是以服务器为准）。

第二，这样做不精确。一旦ntp服务器宕机，跟随它的服务器也就会无法同步时间。与此不同，ntpd不仅能够校准计算机的时间，而且能够校准计算机的时钟。

第三，这样做不够优雅。由于是跳变，而不是使时间变快或变慢，依赖时序的程序会出错（例如，如果ntpdate发现你的时间快了，则可能会经历两个相同的时刻，对某些应用而言，这是致命的）。因而，**唯一一个可以令时间发生跳变的点，是计算机刚刚启动，但还没有启动很多服务的那个时候。其余的时候，理想的做法是使用ntpd来校准时钟，而不是调整计算机时钟上的时间。**

NTPD 在和时间服务器的同步过程中，会把 BIOS 计时器的振荡频率偏差——或者说 Local Clock 的自然漂移(drift)——记录下来。这样即使网络有问题，本机仍然能维持一个相当精确的走时。

