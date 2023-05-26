

[TOC]





# 防火墙工具之iptables实战



# 1 iptables简介

## 1.1 概述

iptables 是 Linux 自带（centos7以后默认使用 firewalld，需要单独安装）的基于包过滤的防火墙工具，其可以对流入、流经、流过服务器的数据包进行精细控制。iptables 主要工作在 OSI 模型的二、三、四层

## 1.2 使用场景

- 做系统主机防火墙，主要使用 filter 表的 INPUT 链
- 做局域网共享上网使用，主要使用 nat 表的 POSTROUTING 链
- 端口或 IP 映射，主要使用 nat 表的 PREROUTING 链
- IP 一对一映射（DMZ）

## 1.3 iptables工作流程

iptables 采用数据包过滤机制工作，因此，它会对请求的数据包的包头数据进行分析，并根据预先设定的规则进行匹配，以决定该数据包是否可以进入主机

- 防火墙是一层一层过滤的，按照配置配置规则的顺序从上到下，从前到后进行过滤
- 如果匹配上了某一条规则，此时数据包就不会继续向下匹配其他规则了
- 如果所有规则中都没有匹配上，则继续向下匹配，直到匹配默认规则，得到明确通过还是阻止
- 防火墙的默认规则是对应链的所有规则执行完毕后才会执行，也就是最后一条规则

## 1.4 iptables的五表五链

### 1.4.1 五表（table）

iptables 中的“表”主要是指明使用 iptables 的功能

- **filter：** 默认表，用于数据包过滤，包含内置的输入、转发和输出链
- **nat：** 地址转换
- **mangle：** 拆解报文做出修改，然后在封装报文（不常用）
- **raw：** 关闭 nat 表上启用的链接追踪机制（不常用）
- **security：** 用于强制访问控制(MAC)网络规则（不常用）



### 1.4.2 五链（chain）

- **PREROUTING：** 作用于路由前
- **INPUT：** 作用于外部进入到本机内
- **FORWARD：** 经由本机向外转发
- **OUTPUT：** 由本机内部向外发出
- **POSTROUTING：** 第二次路由离开本机后

### 1.4.3 报文流向

- **流入：** PREROUTING --> INPUT
- **流出：** OUTPUT --> POSTROUTING
- **转发：** PREROUTING --> FORWARD --> POSTROUTING

### 1.4.4 各功能使用到的链

- **filter：** INPUT、 FORWARD、OUTPUT
- **nat：** PREROUTING(DNAT)、OUTPUT、POSTROUTING(SNAT)
- **mangle：** PREROUTING、INPUT, FORWARD,、OUTPUT、POSTROUTING
- **raw：** PREROUTING、OUTPUT
- **security：** INPUT、FORWARD、OUTPUT



## 1.5 iptables规则

### 1.5.1 规则组成部分

iptables 的规则由报文的匹配条件、匹配到之后处理动作组成

- 匹配条件

  根据协议报文特征指定，包括：基本匹配条件、扩展匹配条件

- 处理动作

  处理动作包括：内建处理机制、自定义处理机制

注意：报文不会经过自定义链，只能在内置链上通过规则进行引用后生效

### 1.5.2 添加规则时的考量点

- 要实现哪种功能

  判断添加在哪张表上

- 报文流经的路径

  判断添加在哪个链上

- 链上规则的次序

  - 同类规则（访问同一应用），匹配范围小的（精确匹配）放上面
  - 不同类规则（访问不同应用），匹配到报文频率较大的放上面
  - 将那些可由一条规则描述的多个规则合并为一个
  - 设置默认策略

- 功能的优先级次序

  raw --> mangle --> nat --> filter







# 2 iptables参数简介

## 2.1  清理参数

```text
-F	清除所有规则，不会处理默认规则 
-X	删除用户自定义的链 
-Z	链的计数器清零
```

## 2.2 查询显示参数

```text
-n	以数字形式显示IP
-L	以列表形式显示所有规则信息
-t	指定表，也可以不指定默认是filter 
-v	表示显示详细规则信息，包含匹配计数器数值信息
--line-number	显示规则序号信息
```

## 2.3 配置参数

- 增删改参数

```text
-t	指定表，也可以不指定默认是filter 
-A	添加规则到指定链的结尾，最后一条 
-I	添加规则到指定链的开头，第一条 
-D	表示删除规则从相应链上
-R	指定将配置好的规则信息进行替换
```

- 规则参数

```text
-P	设置链表的默认规则 
-p	指定协议(all.tcp,udp.icmp)默认all 
-s	指定匹配源地址信息
-d	指定匹配目的地址信息
--dport	指定目的端口 
--sport	指定源端口
-j	处理的行为[ACCPET接收、DROP丢弃、REJECT拒绝]
-i	input	匹配进入的网卡接口, 只能配置在INPUT链上
-o	output	匹配出去的网卡接口, 只能配置在OUTPUT链上
```

- 扩展模块参数

```text
-m	使用扩展模块
-m state		可匹配网络状态 
-m multiport	可匹配多个不连续的端口
-mlimit --limit n/{second/minute/hour}	限制限定时间包的允许通过数量及并发数
```

更多命令可以使用man iptables 或iptables -h



# 2 iptables命令

## 2.1常用命令

### 2.1.1 查询规则

- 语法

  ```bash
  iptables -t 表名 -nvxL --line
  ```

- 参数解释

  > - -t：表名
  > - -n：不解析IP地址
  > - -v：会显示出计数器的信息，数据包的数量和大小
  > - -x：选项表示显示计数器的精确值
  > - –line：显示规则的序号
  > - -L：链名

- 示例

  ```bash
  [root@centos7_11 ~]# iptables -t filter -nvxL --line
  Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
  num      pkts      bytes target     prot opt in     out     source               destination
  1           3      252 ACCEPT     icmp --  *      *       192.168.137.12       0.0.0.0/0
  2          74     4674 ACCEPT     tcp  --  *      *       0.0.0.0/0            192.168.137.11       tcp dpt:22
  3          15     1212 DROP       all  --  *      *       192.168.137.12       0.0.0.0/0
  ```

### 2.1.2 添加规则

- 语法

  ```bash
  # 在最后一条规则后面添加规则
  iptables -t 表名 -A 链名 匹配条件 -j 动作
  
  # 在规则最前面添加规则
  iptables -t 表名 -I 链名 匹配条件 -j 动作
  
  # 在指定位置添加规则
  iptables -t 表名 -I 链名 规则序号 匹配条件 -j 动作
  ```

- 示例

  ```bash
  # 在最后一条规则后面添加规则
  [root@localhost ~]# iptables -t filter -A INPUT -s 192.168.137.12 -j DROP
  
  # 在规则最前面添加规则
  [root@localhost ~]# iptables -t filter -I INPUT -p icmp -s 192.168.137.12 -j ACCEPT
  
  # 在指定位置添加规则
  [root@localhost ~]# iptables -t filter -I INPUT 2 -p tcp -m tcp --dport 22 -d 192.168.137.11 -j ACCEPT
  ```

### 2.1.3 删除规则

- 语法

  ```bash
  # 规则序号删除规则
  iptables -t 表名 -D 链名 规则序号
  
  # 根据具体的匹配条件与动作删除规则
  iptables -t 表名 -D 链名 匹配条件 -j 动作
  
  # 删除指定表的指定链中的所有规则
  iptables -t 表名 -F 链名
  ```

- 示例

  ```bash
  # 规则序号删除规则
  [root@localhost ~]# iptables -t filter -D INPUT 2
  
  # 根据具体的匹配条件与动作删除规则
  [root@localhost ~]# iptables -t filter -D INPUT -p tcp -m tcp --dport 22 -d 192.168.137.11 -j ACCEPT
  
  # 删除指定表的指定链中的所有规则
  [root@localhost ~]# iptables -t filter -F
  ```

### 2.1.4 修改规则

- 语法

  ```bash
  # 修改指定表中指定链的指定规则
  iptables -t 表名 -R 链名 规则序号 规则原本的匹配条件 -j 动作
  
  # 设置指定表的指定链的默认策略
  iptables -t 表名 -P 链名 动作
  ```

- 示例

  ```bash
  # 修改指定表中指定链的指定规则
  [root@localhost ~]# iptables -t filter -R INPUT 1 -p icmp -s 192.168.137.12 -j DROP
  
  # 设置指定表的指定链的默认策略
  [root@localhost ~]# iptables -t filter -P INPUT ACCEPT
  ```

### 2.1.5 保存规则

在 centos7 中，使用 firewall 替代了原来的 iptables service，我们需要通过 yum 安装 iptables 与 iptables-services 即可，在centos7 中安装完 iptables-services 后，即可像 centos6 中一样，通过 `service iptables save` 命令保存规则了，规则保存在 */etc/sysconfig/iptables* 文件中

```bash
# 保存规则
[root@localhost ~]# service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables:[  确定  ]
或者
iptables-save >/etc/sysconfig/iptables

# 查看已保存的规则
[root@localhost ~]# cat /etc/sysconfig/iptables
# Generated by iptables-save v1.4.21 on Thu Dec 29 22:50:25 2022
*filter
:INPUT ACCEPT [33:55612]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [73:5920]
-A INPUT -s 192.168.137.12/32 -p icmp -j DROP
-A INPUT -d 192.168.137.11/32 -p tcp -m tcp --dport 22 -j ACCEPT
-A INPUT -s 192.168.137.12/32 -j DROP
COMMIT
# Completed on Thu Dec 29 22:50:25 2022
```

### 2.1.6 加载规则

我们可以将 */etc/sysconfig/iptables* 中的规则重新载入为当前的 iptables 规则，需要注意的是，未保存入 */etc/sysconfig/iptables* 文件中的修改将会丢失或者被覆盖

使用 `iptables-restore` 命令可以从指定文件中重载规则

```bash
[root@localhost ~]# iptables-restore < /etc/sysconfig/iptables
```



## 2.2 自定义链

如果将所有规则都放在默认链中，假如需要修改某一个规则，则需要从头到尾检查每一条规则，以确保不会将某一服务的规则修改遗漏，如果规则较多，则是一个“大工程”

自定义链可以针对某一服务单独创建一个链，在该链中仅存在对某一服务的规则策略，因此修改相关策略就会相对简单很多，因为该链中不存在其他服务

### 2.2.1 创建自定义链

- 语法

  ```bash
   iptables -t 表名 -N 链名
  ```

- 示例

  ```bash
  [root@localhost ~]# iptables -t filter -N IN_SSH
  ```

### 2.2.2 引用自定义链

- 语法

  ```bash
  iptables -t 表名 -I 默认链 规则 -j 自定义链名
  ```

- 示例

  ```bash
  # 在自定义链中添加规则
  [root@localhost ~]# iptables -t filter -A IN_SSH -s 192.168.137.20 -j DROP
  [root@localhost ~]# iptables -t filter -I IN_SSH  -p icmp -s 192.168.137.20 -j ACCEPT
  
  # 在 INPUT 链中引用刚才创建的自定义链
  [root@localhost ~]# iptables -t filter -I INPUT -j IN_SSH
  
  # 查看规则列表
  [root@localhost ~]# iptables -xnvL --line
  Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
  num      pkts      bytes target     prot opt in     out     source               destination
  1         810    61708 IN_SSH     all  --  *      *       0.0.0.0/0            0.0.0.0/0
  2        1154    77680 ACCEPT     tcp  --  *      *       0.0.0.0/0            192.168.137.11       tcp dpt:22
  
  Chain IN_SSH (1 references)
  num      pkts      bytes target     prot opt in     out     source               destination
  1           3      252 ACCEPT     icmp --  *      *       192.168.137.20       0.0.0.0/0
  2         356    29904 DROP       all  --  *      *       192.168.137.20       0.0.0.0/0
  ```

### 2.2.3 删除自定义链

- 删除自定义链需要满足两个条件

  - 自定义链没有被引用
  - 自定义链中没有任何规则

- 语法

  ```bash
  iptables -t 表名 -X 链名
  ```

- 示例

  ```bash
  # 清空自定义链中的规则
  [root@localhost ~]# iptables -t filter -F IN_SSH
  
  # 删除引用
  [root@localhost ~]# iptables -t filter -D INPUT 1
  
  # 删除自定义链
  [root@localhost ~]# iptables -t filter -X IN_SSH
  ```

### 2.2.4 重命令自定义链

如果自定义链在引用后被重命名，则在引用的规则中会同步修改自定义链的名称为重命名后的名称

- 语法

  ```bash
  iptables -t 表名 -E old-chain-name new-chain-name
  ```

- 示例

  ```bash
  [root@localhost ~]# iptables -t filter -E IN_SSH SSH
  ```



# 3 匹配条件

## 3.1 基本匹配

在匹配条件中，可以使用 `! 条件` 来排除指定条件

- 参数说明

  | 参数        | 解释                                                         |
  | :---------- | :----------------------------------------------------------- |
  | -s ip地址   | 匹配源 ip 地址                                               |
  | -d ip地址   | 匹配目的 ip 地址                                             |
  | -p          | 匹配指定协议 支持匹配的协议包含在 */etc/protocols* 中        |
  | -i 网络接口 | 匹配数据报文的流入接口 该匹配条件仅能用于 PREROUTING、INPUT、FORWARD 链上 |
  | -o 网络接口 | 匹配数据报文的流出接口 该匹配条件仅能用于 FORWARD、OUTPUT、POSTROUTING 链上 |

- 示例

  ```bash
  # 匹配源ip地址
  [root@localhost ~]# iptables -t filter -A INPUT -s 192.168.137.12 -j ACCEPT
  
  # 排除指定源ip地址
  [root@localhost ~]# iptables -t filter -A INPUT ! -s 192.168.137.1 -j DROP
  
  # 匹配目的ip地址
  [root@localhost ~]# iptables -t filter -A INPUT -d 192.168.137.11 -j ACCEPT
  
  # 匹配协议
  [root@localhost ~]# iptables -t filter -A INPUT -p icmp -j ACCEPT
  
  # 匹配流入接口
  [root@localhost ~]# iptables -t filter -A INPUT -i ens33 -j ACCEPT
  
  # 匹配流出接口
  [root@localhost ~]# iptables -t filter -A OUTPUT -o ens33 -j ACCEPT
  ```

## 3.2 扩展匹配

使用扩展匹配的语法为 `-m 模块名 --规则选项` ，在扩展匹配条件中，同样可以使用 `! 条件` 来排除指定条件

### 3.2.1 隐式扩展

使用 `-p 协议名称 `对指定的协议进行的扩展，可省略 `-m` 选项， `-p tcp` 与 `-m tcp` 不冲突，`-p` 用于匹配报文的协议，`-m` 用于指定扩展模块的名称

- 参数说明

  | 参数                   | 解释                                                         |
  | :--------------------- | :----------------------------------------------------------- |
  | –sport PORT[-PORT]     | 源端口，可以是单个端口或连续多个端口                         |
  | –dport PORT[-PORT]     | 目标端口，可以是单个端口或连续多个端口                       |
  | –tcp-flags LIST1 LIST2 | 检查 LIST1 所指明的所有标志位 且这其中 LIST2 所表示出的所有标记位必须为 1，而余下的必须为 0； 没在 LIST1 中指明的，不作检查 可用标志位：SYN、ACK、 FIN、RST、PSH、URG |
  | –syn                   | 用于匹配 tcp 新建连接的请求报文 相当于使用 `–tcp-flags SYN,RST,ACK,FIN SYN` |

- 示例

  ```bash
  [root@localhost ~]# iptables -t filter -A OUTPUT -d 192.168.137.10 -p tcp -m tcp --sport 22 -j DROP
  [root@localhost ~]# iptables -t filter -A INPUT -p tcp -m tcp --dport 22 --tcp-flags SYN,ACK,FIN,RST,URG,PSH SYN -j REJECT
  [root@localhost ~]# iptables -t filter -A INPUT -p tcp -m tcp --dport 22 --tcp-flags ALL SYN -j REJECT
  [root@localhost ~]# iptables -t filter -A INPUT -p tcp -m tcp --dport 22 --syn -j REJECT
  ```

### 3.2.2 显式扩展

显式扩展必须使用 `-m` 显式指明使用的扩展模块，可以使用 `rpm -ql iptables | grep "\.so"` 查看支持的扩展

#### 3.2.2.1 multiport扩展

multiport 扩展可以使用离散方式定义多端口匹配（最多指定 15 个端口）

- 参数说明

  | 参数                              | 解释                           |
  | :-------------------------------- | :----------------------------- |
  | --sports port[,port \| port:port] | 指明多个离散的源端口           |
  | --dports port[,port \| port:port] | 指明多个离散的目标端口         |
  | --ports port[,port \| port:port]  | 指明多个离散的端口（双向控制） |

- 示例

  ```bash
  iptables -I INPUT -s 192.168.137.10 -d 192.168.137.100 -p tcp -m multiport --dports 22,80 -j ACCEPT
  iptables -I OUTPUT -d 192.168.137.100 -s 192.168.137.10 -p tcp -m multiport --sports 22,80 -j ACCEPT
  ```

#### 3.2.2.2 iprange扩展

iprange扩展用于连续的 ip 地址范围时使用

- 参数解释

  | 参数                              | 解释                       |
  | :-------------------------------- | :------------------------- |
  | --src-range 开始ip地址-结束ip地址 | 指定连续的源 ip 地址范围   |
  | --dst-range 开始ip地址-结束ip地址 | 指定连续的目标 ip 地址范围 |

- 示例

  ```bash
  [root@localhost ~]# iptables -I INPUT -d 192.168.137.100 -p tcp -m multiport --dports 22:23,80 -m iprange --src-range 192.168.137.1-192.168.137.10 -j ACCEPT
  [root@localhost ~]# iptables -I OUTPUT -s 192.168.137.100 -p tcp -m multiport --sports 22:23,80 -m iprange --dst-range 192.168.137.1-192.168.137.10 -j ACCEPT		
  ```

#### 3.2.2.3 string扩展

检查报文中出现的字符串

- 参数解释

  | 参数            | 解释                     |
  | :-------------- | :----------------------- |
  | --algo          | 字符串比对算法（必选项） |
  | --string 字符串 | 指定需要匹配的字符串     |

- 示例

  ```bash
  [root@localhost ~]# iptables -I OUTPUT -m string --algo bm --string 'html' -j REJECT
  ```

#### 3.2.2.4 time扩展

根据报文到达的时间与指定的时间范围进行匹配

注意：在使用 `--timestart` 和 `--timestop` 参数时，需要搭配 `--kerneltz` 参数，否则 iptables 默认使用 UTC 时间，会比实际时间晚 8 个小时

- 参数解释

  | 参数        | 解释                                                      |
  | :---------- | :-------------------------------------------------------- |
  | --datestart | 指定日期范围的开始日期，不可取反                          |
  | --datestop  | 指定日期范围的结束日期，不可取反                          |
  | --timestart | 指定时间范围的开始时间，不可取反                          |
  | --timestop  | 指定时间范围的结束时间，不可取反                          |
  | --monthdays | 指定那一天（1-31），可取反                                |
  | --weekdays  | 指定星期几（周一到周日），可取反                          |
  | --kerneltz  | 使用当前系统设置时区的时间，不加此参数，默认使用 UTC 时间 |

- 示例

  ```bash
  [root@localhost ~]# iptables -t filter -I INPUT -p icmp -m time --timestart 16:00:00 --timestop 20:00:00 --kerneltz -j REJECT
  [root@localhost ~]# iptables -t filter -I INPUT -p icmp -m time --datestart 2023-1-5 --datestop 2023-1-6 -j DROP
  [root@localhost ~]# iptables -t filter -I INPUT -p icmp -m time --weekdays 3,5 -j DROP
  [root@localhost ~]# iptables -t filter -I INPUT -p icmp -m time --monthdays 5,7 -j DROP
  ```

#### 3.2.2.5 connlimit扩展

根据每客户端 ip（也可以是地址块）做并发连接数数量匹配

- 参数解释

  | 参数                | 解释                          |
  | :------------------ | :---------------------------- |
  | --connlimit-above n | 连接的数量大于 n 就动作       |
  | --connlimit-upto n  | 连接的数量小于或等于 n 就动作 |

- 示例

  ```bash
  [root@localhost ~]# iptables -I INPUT -p tcp --dport 22 -m connlimit --connlimit-above 2 -j DROP
  ```





# 4 执行的动作

| 参数       | 解释                            |
| :--------- | :------------------------------ |
| ACCEP      | 接受（常用）                    |
| DROP       | 丢弃（常用）                    |
| REJECT     | 拒绝                            |
| RETURN     | 返回调用链                      |
| REDIRECT   | 端口重定向                      |
| LOG        | 记录日志（常用）                |
| MARK       | 做防火墙标记                    |
| DNAT       | 目标地址转换（用于NAT地址转换） |
| SNAT       | 源地址转换（用于NAT地址转换）   |
| MASQUERADE | 地址伪装                        |



# 5 NAT

## 5.1 数据包的流转

- 数据包先经过 NAT 表的 PREROUTING 链
- 经由路由判断确定这个数据包是否要进入本机
- 若不进入本机，再经过 Filter 表的 FORWARD 链
- 通过 NAT 表的 POSTROUTING 链，最后传送出去

## 5.2 NAT分类

- SNAT

  只修改请求报文的源地址，用于私网转公网

- DNAT

  只修改请求报文的目标地址，用于服务映射

## 5.3 nat表相关链

- PREROUTING：用于 DNAT
- OUTPUT
- POSTROUTING：用于 SNAT

## 5.4 SNAT（源地址转换）

### 5.4.1 固定公网ip转换

- 语法

  ```bash
  iptables -t nat -A POSTROUTING -s 内网地址 ! -d 内网地址 -j SNAT --to-source 公网ip
  ```

- 示例

  ```bash
  # 开启Linux转发功能
  [root@localhost ~]# echo "1" > /proc/sys/net/ipv4/ip_forward
  
  # 允许内网中所有数据包通过
  [root@localhost ~]# iptables -A INPUT -i ens33 -j ACCEPT
  
  # SNAT
  [root@localhost ~]# iptables -t nat -A POSTROUTING -s 172.16.10.0/24 ! -d 172.16.10.0/24 -j SNAT --to-source 192.168.137.100
  
  # 查看NAT规则
  [root@localhost ~]# iptables -t nat -nxL --line
  Chain POSTROUTING (policy ACCEPT)
  num  target     prot opt source               destination
  1    SNAT       all  --  172.16.10.0/24      !172.16.10.0/24       to:192.168.137.100
  ```

### 5.4.2 动态公网ip（地址伪装）

- 语法

  ```bash
  iptables -t nat -A POSTROUTING -s 内网地址 ! -d 内网地址 -j MASQUERADE
  ```

- 示例

  ```bash
  # SNAT
  [root@localhost ~]# iptables -t nat -A POSTROUTING -s 172.16.10.0/24 ! -d 172.16.10.0/24 -j MASQUERADE
  
  # 查看NAT规则
  [root@localhost ~]# iptables -t nat -nxL --line
  Chain POSTROUTING (policy ACCEPT)
  num  target     prot opt source               destination
  1   MASQUERADE  all  --  172.16.10.0/24      !172.16.10.0/24
  ```

## 5.5 DNAT（目标地址转换）

- 语法

  ```bash
  iptables -t nat -A PREROUTING -d EXT_IP -p tcp|udp --dport PORT -j DNAT --to-destination INTER_SERVER_IP[:PORT]
  ```

- 示例

  ```bash
  # DNAT
  [root@localhost ~]# iptables -t nat -A PREROUTING -d 192.168.137.100 -p tcp --dport 80 -j DNAT --to-destination 172.16.10.10:80
  
  # 查看NAT规则
  [root@localhost ~]# iptables -t nat -nxL --line
  Chain PREROUTING (policy ACCEPT)
  num  target     prot opt source               destination
  1    DNAT       tcp  --  0.0.0.0/0            192.168.137.100      tcp dpt:80 to:172.16.10.10:80
  ```

## 5.6 链接跟踪表容量和超时时间设置

```bash
vim /etc/sysctl.conf
# 增加 ip_conntrack_max 值
net.ipv4.ip_conntrack_max = 393216
net.ipv4.netfilter.ip_conntrack_max = 393216

# 降低 ip_conntrack timeout 时间
net.ipv4.netfilter.ip_conntrack_tcp_timeout_established = 300
net.ipv4.netfilter.ip_conntrack_tcp_timeout_time_wait = 120
net.ipv4.netfilter.ip_conntrack_tcp_timeout_close_wait = 60
net.ipv4.netfilter.ip_conntrack_tcp_timeout_fin_wait = 120
```











# 6 案例

**1.拒绝进入防火墙的所有ICMP协议数据包**

```shell
iptables -I INPUT -p icmp -j REJECT
```

**2.允许防火墙转发除ICMP协议以外的所有数据包**

```shell
iptables -A FORWARD -p ! icmp -j ACCEPT
```

说明：使用“！”可以将条件取反。

**3.拒绝转发来自192.168.1.10主机的数据，允许转发来自192.168.0.0/24网段的数据**

```shell
iptables -A FORWARD -s 192.168.1.11 -j REJECT
iptables -A FORWARD -s 192.168.0.0/24 -j ACCEPT
```

说明：注意要把拒绝的放在前面不然就不起作用了啊。

**4.丢弃从外网接口（eth1）进入防火墙本机的源地址为私网地址的数据包**

```shell
iptables -A INPUT -i eth1 -s 192.168.0.0/16 -j DROP
iptables -A INPUT -i eth1 -s 172.16.0.0/12 -j DROP
iptables -A INPUT -i eth1 -s 10.0.0.0/8 -j DROP
```

**5.封堵网段（192.168.1.0/24），两小时后解封。**

```shell
iptables -I INPUT -s 10.20.30.0/24 -j DROP

iptables -I FORWARD -s 10.20.30.0/24 -j DROP

at now 2 hours 
at> iptables -D INPUT 1 
at> iptables -D FORWARD 1
```

说明：这个策略咱们借助crond计划任务来完成，就再好不过了。

```shell
[1]   Stopped     at now 2 hours
```

**6.只允许管理员从202.13.0.0/16网段使用SSH远程登录防火墙主机。**

```shell
iptables -A INPUT -p tcp --dport 22 -s 202.13.0.0/16 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP
```

说明：这个用法比较适合对设备进行远程管理时使用，比如位于分公司中的SQL服务器需要被总公司的管理员管理时。

**7、允许本机开放从TCP端口20-1024提供的应用服务。**

```shell
iptables -A INPUT -p tcp --dport 20:1024 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 20:1024 -j ACCEPT
```

**8.允许转发来自192.168.0.0/24局域网段的DNS解析请求数据包。**

```shell
iptables -A FORWARD -s 192.168.0.0/24 -p udp --dport 53 -j ACCEPT
iptables -A FORWARD -d 192.168.0.0/24 -p udp --sport 53 -j ACCEPT
```

**9.禁止其他主机ping防火墙主机，但是允许从防火墙上ping其他主机**

```shell
iptables -I INPUT -p icmp --icmp-type Echo-Request -j DROP
iptables -I INPUT -p icmp --icmp-type Echo-Reply -j ACCEPT
iptables -I INPUT -p icmp --icmp-type destination-Unreachable -j ACCEPT
```

**10.禁止转发来自MAC地址为00：0C：29：27：55：3F的和主机的数据包**

```shell
iptables -A FORWARD -m mac --mac-source 00:0c:29:27:55:3F -j DROP
```

说明：iptables中使用“-m 模块关键字”的形式调用显示匹配。咱们这里用“-m mac –mac-source”来表示数据包的源MAC地址。

**11.允许防火墙本机对外开放TCP端口20、21、25、110以及被动模式FTP端口1250-1280**

```shell
iptables -A INPUT -p tcp -m multiport --dport 20,21,25,110,1250:1280 -j ACCEPT
```

说明：这里用“-m multiport –dport”来指定目的端口及范围

**12.禁止转发源IP地址为192.168.1.20-192.168.1.99的TCP数据包。**

```shell
iptables -A FORWARD -p tcp -m iprange --src-range 192.168.1.20-192.168.1.99 -j DROP
```

说明：此处用“-m –iprange –src-range”指定IP范围。

**13.禁止转发与正常TCP连接无关的非—syn请求数据包。**

```shell
iptables -A FORWARD -m state --state NEW -p tcp ! --syn -j DROP
```

说明：“-m state”表示数据包的连接状态，“NEW”表示与任何连接无关的，新的嘛！

**14.拒绝访问防火墙的新数据包，但允许响应连接或与已有连接相关的数据包**

```shell
iptables -A INPUT -p tcp -m state --state NEW -j DROP
iptables -A INPUT -p tcp -m state --state ESTABLISHED,RELATED -j ACCEPT
```

说明：“ESTABLISHED”表示已经响应请求或者已经建立连接的数据包，“RELATED”表示与已建立的连接有相关性的，比如FTP数据连接等。

**15.只开放本机的web服务（80）、FTP(20、21、20450-20480)，放行外部主机发往服务器其它端口的应答数据包，将其他入站数据包均予以丢弃处理。**

```shell
iptables -I INPUT -p tcp -m multiport --dport 20,21,80 -j ACCEPT
iptables -I INPUT -p tcp --dport 20450:20480 -j ACCEPT
iptables -I INPUT -p tcp -m state --state ESTABLISHED -j ACCEPT
iptables -P INPUT DROP
```



**16.限制只有10.220.5.188可以连接ssh**

```sh
iptables -A INPUT -s 10.220.5.188 -p tcp --dport 22 -j ACCEPT
```

**17.允许10.220.5.188访问本机22，80，3306，100到200的端口**

```sh
iptables -A INPUT -p tcp -m multiport --dport 22,80,3306,100:200 -m state --state NEW,ESTABLISHED -j ACCEPT
```



**18.只允许ip地址10.220.5.10至10.220.5.20之间的主机访问本机的80端口**

```sh
iptables -A INPUT -p tcp -m iprange --src-range 10.220.5.10-10.220.5.20 --dport 80  -j ACCEPT
```



**19.防暴力破解&DOS攻击，限制请求登录22端口的频率**

```sh
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m limit --limit 10/minute --limit-burst 20 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -m state --state ESTABLISHED -j ACCEPT
```



**20.限制只能从10.220.5.188登录后台界面（admin.php）**

```sh
iptables -A INPUT -s 10.220.5.188 -m string --algo bm --string "admin.php" -j ACCEPT
```



**21.限制每个用户只能同时登录5个ssh**

```sh
iptables -A INPUT -p tcp -m connlimit ! --connlimit-above 5 --dport 22  -j ACCEPT
```

**22.限制每个客户端只能与80端口并发连接10个链接**

```sh
iptables -A INPUT -p tcp --dport 80 -m state --state NEW,ESTABLISHED -m connlimit ！--connlimit-above 10 -j ACCEPT
```

**23.指定在1h只登录达到5次之上的，该次链接请求会被丢弃**

```sh
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --name loginSSH --update --seconds 3600 --hitcount 5 -j DROP
```





































