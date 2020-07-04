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

注意：此处需要等待服务端几分钟。

添加到定时任务

```shell
cat >>/var/spool/cron/root<<EOF
#crond m01
*/5 * * * * /usr/sbin/ntpdate 10.0.0.41 >/dev/null 2>&1
EOF
```





# 3.1 无外网时间同步

服务端

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



修改/etc/ntp.conf

1. 注释掉原来的restrict default ignore这一行，这一行本身是不响应任何的ntp更新请求，

    其实也就是禁用了本机的ntp server的功能，所以需要注释掉。
    
    2. 加入下面3行： 
    
    restrict 10.10.10.0 mask 255.255.255.0 nomodify notrap
    （注释:用于让10.10.10.0/24网段上的机器能和本机做时间同步）
    server 127.127.1.0 # local clock
    fudge 127.127.1.0 stratum 10
    
    后两行是让本机的ntpd和本地硬件时间同步。
    
    3./etc/init.d/ntpd restart或者 service ntpd restart
    4.chkconfig ntpd on 设置开机自启动
    5.修改iptables配置，将tcp和udp 123端口开放，这是ntp需要的端口，在/etc/services中可以查到这个端口






