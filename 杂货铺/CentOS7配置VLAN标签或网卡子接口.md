



# CentOS7配置VLAN标签或网卡子接口



1、确认系统加载了802.1Q模块，并配置主网卡

```bash
[root@localhost ~]# modprobe --first-time 8021q
modprobe: ERROR: could not insert '8021q': Module already in kernel
[root@localhost ~]# lsmod |grep 8021
8021q                  33159  0
garp                   14384  1 8021q
mrp                    18542  1 8021q

#主网卡配置（无特殊，只要开机自启）
[root@localhost ~]# vim /etc/sysconfig/network-scripts/ifcfg-enp8s0f0
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
DEFROUTE=yes
NAME=enp8s0f0
UUID=8228997b-5f76-4dde-8698-30f67e6ed5c8
DEVICE=enp8s0f0
ONBOOT=yes
```

2、创建带vlan标签的网卡文件

```bash
[root@localhost ~]# vim /etc/sysconfig/network-scripts/ifcfg-enp8s0f0.237
BOOTPROTO=none
DEFROUTE=yes
DEVICE=enp8s0f0.237
ONBOOT=yes
VLAN=yes
IPADDR=10.159.237.1
NETMASK=255.255.255.0
GATEWAY=10.159.237.254
DNS1=10.150.0.251
DNS2=10.150.0.241
#创建带vlan的网卡设备（vlan标签237 对应网卡名字为ifcfg-enp8s0f0.237 ）
```



3、重启网络服务或服务器，并验证vlan标签是否可使用

```shell
[root@localhost ~]# systemctl restart network
[root@localhost ~]# cat /proc/net/vlan/enp8s0f0.237
enp8s0f0.237  VID: 237	 REORDER_HDR: 1  dev->priv_flags: 1
         total frames received          223
          total bytes received        17659
      Broadcast/Multicast Rcvd            0

      total frames transmitted          205
       total bytes transmitted        31270
Device: enp8s0f0
INGRESS priority mappings: 0:0  1:0  2:0  3:0  4:0  5:0  6:0 7:0
 EGRESS priority mappings:

```