[TOC]







# Linux系统性能分析之sysstat



# 简介

Sysstat是一个软件工具集，包含监测系统性能及效率的一组工具。这些工具能够收集系统性能数据，比如CPU使用率、硬盘和网络吞吐量数据，这些数据的收集和分析，有利于判断系统是否正常运行，是提高系统运行效率、安全运行服务器的得力助手。安装后包含如下命令：

- iostat-提供CPU统计，存储I/O统计（磁盘设备，分区及网络文件系统）
- mpstat-提供单个或组合CPU相关统计
- pidstat-提供Linux进程级别统计：I/O、CPU、内存等
- sar-收集、报告、保存系统活动信息：CPU、内存、磁盘、中断、网络接口、TTY、内核表等
- sadc-系统活动数据收集器，作为sar后端使用
- sa1-收集系统活动日常数据，并以二进制格式存储，它作为sadc的工具的前端，可以通过cron来调用
- sa2-生成系统每日活动报告，同样可作为sadc的工具的前端，可以通过cron来调用
- sadf-可以以CSV、XML等格式显示sar收集的性能数据，这样可以非常方便的将系统数据导入到数据库中，或导入到Excel中生成图表
- nfsiostat-提供NFS I/O统计
- cifsiostat-提供CIFS统计



# 安装

```
yum -y install sysstat
```

依赖 `lm_sensors-libs`。

















# sar使用

sar（System Activity Reporter系统活动情况报告）是目前 Linux 上最为全面的系统性能分析工具之一，可以从多方面对系统的活动进行报告，包括：文件的读写情况、系统调用的使用情况、磁盘I/O、CPU效率、内存使用状况、进程活动及IPC有关的活动等。

## sar命令常用格式

sar [options] [-A] [-o file] t [n]

其中：

t为采样间隔，n为采样次数，默认值是1；

-o file表示将命令结果以二进制格式存放在文件中，file 是文件名。

options 为命令行选项，sar命令常用选项如下：

-A：所有报告的总和

-u：输出CPU使用情况的统计信息

-v：输出inode、文件和其他内核表的统计信息

-d：输出每一个块设备的活动信息

-r：输出内存和交换空间的统计信息

-b：显示I/O和传送速率的统计信息

-a：文件读写情况

-c：输出进程统计信息，每秒创建的进程数

-R：输出内存页面的统计信息

-y：终端设备活动情况

-w：输出系统交换活动信息



## CPU监控

### 统计cpu的使用情况

例如，每10秒采样一次，连续采样3次，观察CPU 的使用情况，并将采样结果以二进制形式存入当前目录下的文件test中，需键入如下命令：

```
[root@ldap-server ~]# sar -u -o test 10 3
Linux 3.10.0-693.el7.x86_64 (ldap-server) 	03/08/2022 	_x86_64_	(4 CPU)

10:46:38 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
10:46:48 AM     all      0.03      0.00      0.05      0.00      0.00     99.92
10:46:58 AM     all      0.10      0.00      0.08      0.00      0.00     99.82
10:47:08 AM     all      0.03      0.00      0.03      0.00      0.00     99.95
Average:        all      0.05      0.00      0.05      0.00      0.00     99.90

```

输出项说明：

- CPU：all 表示统计信息为所有 CPU 的平均值。

- %user：显示在用户级别(application)运行使用 CPU 总时间的百分比。
- %nice：显示在用户级别，用于nice操作，所占用 CPU 总时间的百分比。
- %system：在核心级别(kernel)运行所使用 CPU 总时间的百分比。
- %iowait：显示用于等待I/O操作占用 CPU 总时间的百分比。
- %steal：管理程序(hypervisor)为另一个虚拟进程提供服务而等待虚拟 CPU 的百分比。
- %idle：显示 CPU 空闲时间占用 CPU 总时间的百分比。

（1）若 %iowait 的值过高，表示硬盘存在I/O瓶颈，即磁盘IO无法满足业务需求

（2）若 %idle 的值高但系统响应慢时，有可能是 CPU 等待分配内存，需要结合内存使用等情况判断

（3）若 %idle 的值持续低于1，则系统的 CPU 处理能力相对较低，表明系统中最需要解决的资源是 CPU 。

如果要查看二进制文件test中的内容，需键入如下sar命令：

```bash
sar -u -f test
```



## 查看每个cpu的使用状态

```
[root@ldap-server ~]# sar -p 1 3
Linux 3.10.0-693.el7.x86_64 (ldap-server) 	03/08/2022 	_x86_64_	(4 CPU)

11:01:26 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
11:01:27 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
11:01:28 AM     all      0.00      0.00      0.25      0.00      0.00     99.75
11:01:29 AM     all      0.00      0.00      0.25      0.00      0.00     99.75
Average:        all      0.00      0.00      0.17      0.00      0.00     99.83

```

输出项说明：

- CPU 所有CPU的统计
- %user 用户态的CPU使用统计
- %nice 更改过优先级的进程的CPU使用统计
- %iowait CPU等待IO数据的百分比
- %steal 虚拟机的vCPU占用的物理CPU的百分比
- %idle 空闲的CPU百分比



## 进程队列长度和平均负载监控

例如，每10秒采样一次，连续采样3次，监控进程队列长度和平均负载状态：

```
sar -q 10 3

屏幕显示如下：
19:25:50 runq-sz plist-sz ldavg-1 ldavg-5 ldavg-15
19:26:00 0 259 0.00 0.00 0.00
19:26:10 0 259 0.00 0.00 0.00
19:26:20 0 259 0.00 0.00 0.00
Average: 0 259 0.00 0.00 0.00
```

输出项说明：

- runq-sz：运行队列的长度（等待运行的进程数）
- plist-sz：进程列表中进程（processes）和线程（threads）的数量
- ldavg-1：最后1分钟的系统平均负载（System load average）
- ldavg-5：过去5分钟的系统平均负载
- ldavg-15：过去15分钟的系统平均负载





## Inode、文件和其他内核表监控

例如，每10秒采样一次，连续采样3次，观察核心表的状态，需键入如下命令：

```
$ sar -v 10 3
17:10:49 dentunusd file-nr inode-nr pty-nr
17:10:59 6301 5664 12037 4
17:11:09 6301 5664 12037 4
17:11:19 6301 5664 12037 4
Average: 6301 5664 12037 4
```

输出项说明：

- dentunusd：目录高速缓存中未被使用的条目数量
- file-nr：文件句柄（file handle）的使用数量
- inode-nr：索引节点句柄（inode handle）的使用数量
- pty-nr：使用的pty数量



## 内存和交换空间监控

例如，每10秒采样一次，连续采样3次，监控内存分页：

```
sar -r 10 3
```

输出项说明：

- kbmemfree：这个值和free命令中的free值基本一致,所以它不包括buffer和cache的空间.
- kbmemused：这个值和free命令中的used值基本一致,所以它包括buffer和cache的空间.
- %memused：这个值是kbmemused和内存总量(不包括swap)的一个百分比.
- kbbuffers和kbcached：这两个值就是free命令中的buffer和cache.
- kbcommit：保证当前系统所需要的内存,即为了确保不溢出而需要的内存(RAM+swap).
- %commit：这个值是kbcommit与内存总量(包括swap)的一个百分比.

查看交换空间swap的统计信息

```
sar -W
```

输出项说明：

- pswpin/s  每秒从交换分区到系统的交换页面（swap page）数量
- pswpott/s 每秒从系统交换到swap的交换页面（swap page）的数量



## 内存分页监控

例如，每10秒采样一次，连续采样3次，监控内存分页：

```
sar -B 10 3
```

输出项说明：

- pgpgin/s：表示每秒从磁盘或SWAP置换到内存的字节数(KB)
- pgpgout/s：表示每秒从内存置换到磁盘或SWAP的字节数(KB)
- fault/s：每秒钟系统产生的缺页数,即主缺页与次缺页之和(major + minor)
- majflt/s：每秒钟产生的主缺页数.
- pgfree/s：每秒被放入空闲队列中的页个数
- pgscank/s：每秒被kswapd扫描的页个数
- pgscand/s：每秒直接被扫描的页个数
- pgsteal/s：每秒钟从cache中被清除来满足内存需要的页个数
- %vmeff：每秒清除的页(pgsteal)占总扫描页(pgscank+pgscand)的百分比



## I/O和传递速率监控

例如，每10秒采样一次，连续采样3次，报告缓冲区的使用情况，需键入如下命令：



```
$ sar -b 10 3
18:51:05 tps rtps wtps bread/s bwrtn/s
18:51:15 0.00 0.00 0.00 0.00 0.00
18:51:25 1.92 0.00 1.92 0.00 22.65
18:51:35 0.00 0.00 0.00 0.00 0.00
Average: 0.64 0.00 0.64 0.00 7.59
```

输出项说明：

- tps：每秒钟物理设备的 I/O 传输总量
- rtps：每秒钟从物理设备读入的数据总量
- wtps：每秒钟向物理设备写入的数据总量
- bread/s：每秒钟从物理设备读入的数据量，单位为 块/s
- bwrtn/s：每秒钟向物理设备写入的数据量，单位为 块/s



## 网络监控

```
sar -n
```

选项使用6个不同的开关：DEV，EDEV，NFS，NFSD，SOCK，IP，EIP，ICMP，EICMP，TCP，ETCP，UDP，SOCK6，IP6，EIP6，ICMP6，EICMP6和UDP6 ，DEV显示网络接口信息，EDEV显示关于网络错误的统计数据，NFS统计活动的NFS客户端的信息，NFSD统计NFS服务器的信息，SOCK显示套接字信息，ALL显示所有5个开关。它们可以单独或者一起使用。 

每间隔1秒统计一次，总计统计1次，下面的average是在多次统计后的平均值：

```
[root@ldap-server ~]# sar -n DEV 1 1
Linux 3.10.0-693.el7.x86_64 (ldap-server) 	03/08/2022 	_x86_64_	(4 CPU)

11:28:59 AM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
11:29:00 AM      eth0     22.00      1.00      1.45      0.12      0.00      0.00      0.00
11:29:00 AM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00

Average:        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
Average:         eth0     22.00      1.00      1.45      0.12      0.00      0.00      0.00
Average:           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00

```

输出项说明：

- IFACE 本地网卡接口的名称
- rxpck/s 每秒钟接受的数据包
- txpck/s 每秒钟发送的数据库
- rxKB/S 每秒钟接受的数据包大小，单位为KB
- txKB/S 每秒钟发送的数据包大小，单位为KB
- rxcmp/s 每秒钟接受的压缩数据包
- txcmp/s 每秒钟发送的压缩包
- rxmcst/s 每秒钟接收的多播数据包  



### 统计网络设备通信失败信息

```
[root@ldap-server ~]# sar -n EDEV  1 1
Linux 3.10.0-693.el7.x86_64 (ldap-server) 	03/08/2022 	_x86_64_	(4 CPU)

11:30:17 AM     IFACE   rxerr/s   txerr/s    coll/s  rxdrop/s  txdrop/s  txcarr/s  rxfram/s  rxfifo/s  txfifo/s
11:30:18 AM      eth0      0.00      0.00      0.00      1.00      0.00      0.00      0.00      0.00      0.00
11:30:18 AM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

Average:        IFACE   rxerr/s   txerr/s    coll/s  rxdrop/s  txdrop/s  txcarr/s  rxfram/s  rxfifo/s  txfifo/s
Average:         eth0      0.00      0.00      0.00      1.00      0.00      0.00      0.00      0.00      0.00
Average:           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
```

输出项说明：

- IFACE 网卡名称
- rxerr/s 每秒钟接收到的损坏的数据包
- txerr/s 每秒钟发送的数据包错误数
- coll/s 当发送数据包时候，每秒钟发生的冲撞（collisions）数，这个是在半双工模式下才有
- rxdrop/s 当由于缓冲区满的时候，网卡设备接收端每秒钟丢掉的网络包的数目
- txdrop/s 当由于缓冲区满的时候，网络设备发送端每秒钟丢掉的网络包的数目
- txcarr/s  当发送数据包的时候，每秒钟载波错误发生的次数
- rxfram  在接收数据包的时候，每秒钟发生的帧对其错误的次数
- rxfifo   在接收数据包的时候，每秒钟缓冲区溢出的错误发生的次数
- txfifo   在发生数据包 的时候，每秒钟缓冲区溢出的错误发生的次数



### 统计socket连接信息

```
[root@ldap-server ~]# sar -n SOCK 1 1
Linux 3.10.0-693.el7.x86_64 (ldap-server) 	03/08/2022 	_x86_64_	(4 CPU)

11:32:37 AM    totsck    tcpsck    udpsck    rawsck   ip-frag    tcp-tw
11:32:38 AM       234         7         0         0         0         0
Average:          234         7         0         0         0         0

```

输出项说明：

- totsck 当前被使用的socket总数
- tcpsck 当前正在被使用的TCP的socket总数
- udpsck  当前正在被使用的UDP的socket总数
- rawsck 当前正在被使用于RAW的skcket总数
- if-frag  当前的IP分片的数目
- tcp-tw TCP套接字中处于TIME-WAIT状态的连接数量

如果你使用FULL关键字，相当于上述DEV、EDEV和SOCK三者的综合。



### TCP连接的统计

```
[root@ldap-server ~]# sar -n TCP 1 3
Linux 3.10.0-693.el7.x86_64 (ldap-server) 	03/08/2022 	_x86_64_	(4 CPU)

11:33:43 AM  active/s passive/s    iseg/s    oseg/s
11:33:44 AM      0.00      0.00      1.00      1.00
11:33:45 AM      0.00      0.00      1.00      1.00
11:33:46 AM      0.00      0.00      1.00      1.00
Average:         0.00      0.00      1.00      1.00

```

输出项说明：

- active/s 新的主动连接
- passive/s 新的被动连接
- iseg/s 接受的段
- oseg/s 输出的段



**sar -n 使用总结**

```bsh
-n DEV ： 网络接口统计信息。
-n EDEV ： 网络接口错误。
-n IP ： IP数据报统计信息。
-n EIP ： IP错误统计信息。
-n TCP ： TCP统计信息。
-n ETCP ： TCP错误统计信息。
-n SOCK ： 套接字使用。
```



## 设备使用情况监控

例如，每10秒采样一次，连续采样3次，报告设备使用情况，需键入如下命令：

```bash
[root@ldap-server ~]# sar -d -p 10 3
Linux 3.10.0-693.el7.x86_64 (ldap-server) 	03/08/2022 	_x86_64_	(4 CPU)

11:43:52 AM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
11:44:02 AM       vda      0.20      0.00      1.60      8.00      0.00      0.00      0.00      0.00
11:44:02 AM       vdb      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
11:44:02 AM centos-root      0.20      0.00      1.60      8.00      0.00      0.00      0.00      0.00
11:44:02 AM centos-swap      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
11:44:02 AM centos-home      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

11:44:02 AM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
11:44:12 AM       vda      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
11:44:12 AM       vdb      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
11:44:12 AM centos-root      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
11:44:12 AM centos-swap      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
11:44:12 AM centos-home      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

11:44:12 AM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
11:44:22 AM       vda      1.20      0.00     13.80     11.50      0.01      7.58      7.00      0.84
11:44:22 AM       vdb      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
11:44:22 AM centos-root      1.20      0.00     13.80     11.50      0.01      7.75      7.00      0.84
11:44:22 AM centos-swap      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
11:44:22 AM centos-home      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

Average:          DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
Average:          vda      0.47      0.00      5.13     11.00      0.00      6.50      6.00      0.28
Average:          vdb      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
Average:    centos-root      0.47      0.00      5.13     11.00      0.00      6.64      6.00      0.28
Average:    centos-swap      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
Average:    centos-home      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

```

输出项说明：

参数-p可以打印出sda,hdc等磁盘设备名称,如果不用参数-p,设备节点则有可能是dev8-0,dev22-0

tps:每秒从物理磁盘I/O的次数.多个逻辑请求会被合并为一个I/O磁盘请求,一次传输的大小是不确定的.

rd_sec/s:每秒读扇区的次数.

wr_sec/s:每秒写扇区的次数.

avgrq-sz:平均每次设备I/O操作的数据大小(扇区).

avgqu-sz:磁盘请求队列的平均长度.

await:从请求磁盘操作到系统完成处理,每次请求的平均消耗时间,包括请求队列等待时间,单位是毫秒(1秒=1000毫秒).

svctm:系统处理每次请求的平均时间,不包括在请求队列中消耗的时间.

%util:I/O请求占CPU的百分比,比率越大,说明越饱和.

（1）avgqu-sz 的值较低时，设备的利用率较高。

（2）当%util的值接近 100% 时，表示设备带宽已经占满。

**对于磁盘 IO 性能，一般有如下评判标准：**

正常情况下 svctm 应该是小于 await 值的，而 svctm 的大小和磁盘性能有关，CPU 、内存的负荷也会对 svctm 值造成影响，过多的请求也会间接的导致 svctm 值的增加。

await 值的大小一般取决与 svctm 的值和 I/O 队列长度以 及I/O 请求模式，如果 svctm 的值与 await 很接近，表示几乎没有 I/O 等待，磁盘性能很好，如果 await 的值远高于 svctm 的值，则表示 I/O 队列等待太长，系统上运行的应用程序将变慢，此时可以通过更换更快的硬盘来解决问题。

%util 项的值也是衡量磁盘 I/O 的一个重要指标，如果 %util 接近 100% ，表示磁盘产生的 I/O 请求太多，I/O 系统已经满负荷的在工作，该磁盘可能存在瓶颈。长期下去，势必影响系统的性能，可以通过优化程序或者通过更换更高、更快的磁盘来解决此问题。





**常用命令汇总，因版本和平台不同，有部分命令可能没有或显示结果不一致：**

```
默认监控: sar 5 5     //  CPU和IOWAIT统计状态 
(1) sar -b 5 5        // IO传送速率
(2) sar -B 5 5        // 页交换速率
(3) sar -c 5 5        // 进程创建的速率
(4) sar -d 5 5        // 块设备的活跃信息
(5) sar -n DEV 5 5    // 网路设备的状态信息
(6) sar -n SOCK 5 5   // SOCK的使用情况
(7) sar -n ALL 5 5    // 所有的网络状态信息
(8) sar -P ALL 5 5    // 每颗CPU的使用状态信息和IOWAIT统计状态 
(9) sar -q 5 5        // 队列的长度（等待运行的进程数）和负载的状态
(10) sar -r 5 5       // 内存和swap空间使用情况
(11) sar -R 5 5       // 内存的统计信息（内存页的分配和释放、系统每秒作为BUFFER使用内存页、每秒被cache到的内存页）
(12) sar -u 5 5       // CPU的使用情况和IOWAIT信息（同默认监控）
(13) sar -v 5 5       // inode, file and other kernel tablesd的状态信息
(14) sar -w 5 5       // 每秒上下文交换的数目
(15) sar -W 5 5       // SWAP交换的统计信息(监控状态同iostat 的si so)
(16) sar -x 2906 5 5  // 显示指定进程(2906)的统计信息，信息包括：进程造成的错误、用户级和系统级用户CPU的占用情况、运行在哪颗CPU上
(17) sar -y 5 5       // TTY设备的活动状态
(18) 将输出到文件(-o)和读取记录信息(-f)
sar也可以监控非实时数据，通过cron周期的运行到指定目录下
例如:我们想查看本月27日,从0点到23点的内存资源.
sa27就是本月27日,指定具体的时间可以通过-s(start)和-e(end)来指定.
sar -f /var/log/sa/sa27 -s 00:00:00 -e 23:00:00 -r
```









# man-sysstat

## 1. sysstat

Sysstat-资源监视工具的集合：iostat，isag，mpstat，pidstat，sadf，sar。
http://sebastien.godard.pagesperso-orange.fr/
https://github.com/sysstat/sysstat
注: 本文是sysstat相关命令的man *文档的部分摘录翻译

### 1.1 sysstat 包含的命令

| Command    | DESCRIPTION                                                  | cn                                                           |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| iostat     | Report Central Processing Unit (CPU) statistics and input/output statistics for devices and partitions | 报告设备和分区的中央处理单元(CPU)统计信息以及输入/输出统计信息 |
| mpstat     | Report processors related statistics                         | 报告处理器相关统计                                           |
| pidstat    | Report statistics for Linux tasks                            | 报告Linux任务的统计信息                                      |
| sar        | Collect, report, or save system activity information         | 收集，报告或保存系统活动信息                                 |
| sadf       | Display data collected by sar in multiple formats            | 以多种格式显示sar收集的数据                                  |
| cifsiostat | Report CIFS statistics                                       | 报告CIFS统计信息                                             |
| tapestat   | Report tape statistics                                       | 报告磁带统计                                                 |

### 1.2 命令相关的文件

| Command    | FILE                                                         | DESCRIPTION                                                  | cn                                                           |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| sysstat    | /etc/conf.d/sysstat                                          |                                                              | 配置文件                                                     |
| cifsiostat | /proc/fs/cifs/Stats                                          | contains CIFS statistics.                                    | CIFS统计                                                     |
| iostat     | /proc/stat /proc/uptime /proc/diskstats /sys /proc/self/mountstats /dev/disk | contains system statistics. contains system uptime. contains disks statistics. contains statistics for block devices. contains statistics for network filesystems. contains persistent device names. | 系统统计 正常运行时间 磁盘统计 块设备统计 网络文件系统统计 持久性设备名称 |
| mpstat     | /proc                                                        | contains various files with system statistics.               |                                                              |
| pidstat    | /proc                                                        | 包含具有系统统计信息的各种文件。                             |                                                              |
| sadf       | /var/log/sa/saDD /var/log/sa/saYYYYMMDD                      | 标准系统活动每日数据文件及其默认位置。 YYYYMMDD: year, month and day. |                                                              |
| sar        | /var/log/sa/saDD /var/log/sa/saYYYYMMDD /proc and /sys       | The standard system activity daily data files and their default location. Contain various files with system statistics. |                                                              |
| tapestat   | /sys/class/scsi_tape/st<num>/stats/* /proc/uptime            | Statistics files for tape devices. contains system uptime.   | 磁带设备统计文件 正常运行时间                                |







### 1.3 EXAMPLES

| EXAMPLES                              | DESCRIPTION                                                  | cn                                                           |
| ------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| iostat                                | Display a single history since boot report for all CPU and Devices. | 自启动报告以来，显示所有CPU和设备的单个历史记录。            |
| iostat -d 2                           | Display a continuous device report at two second intervals.  | 每两秒显示一次连续的设备报告。                               |
| iostat -d 2 6                         | Display six reports at two second intervals for all devices. | 对于所有设备，每隔两秒显示六个报告。                         |
| iostat -x sda sdb 2 6                 | Display six reports of extended statistics at two second intervals for devices sda and sdb. | 以两秒为间隔显示设备sda和sdb的六个扩展统计信息报告。         |
| iostat -p sda 2 6                     | Display six reports at two second intervals for device sda and all its partitions (sda1, etc.) | 以两秒的间隔显示六个关于设备sda及其所有分区（sda1等）的报告。 |
| mpstat 2 5                            | Display five reports of global statistics among all processors at two second intervals. | 每隔两秒钟显示所有处理器之间的五个全局统计报告。             |
| mpstat -P ALL 2 5                     | Display five reports of statistics for all processors at two second intervals. | 以两秒为间隔显示所有处理器的五个统计报告。                   |
| pidstat 2 5                           | Display five reports of CPU statistics for every active task in the system at two second intervals. | 以两秒为间隔显示系统中每个活动任务的五个CPU统计信息报告。    |
| pidstat -r -p 1643 2 5                | Display five reports of page faults and memory statistics for PID 1643 at two second intervals. | 每两秒显示一次PID的页面错误和内存统计信息的五个报告。        |
| pidstat -C "fox\|bird" -r -p ALL      | Display global page faults and memory statistics for all the processes whose command name includes the string "fox" or "bird". | 显示所有命令名称包含字符串“ fox”或“ bird”的进程的全局页面错误和内存统计信息。 |
| pidstat -T CHILD -r 2 5               | Display five reports of page faults statistics at two second intervals for the child processes of all tasks in the system. Only child processes with non-zero statistics values are displayed. | 对于系统中所有任务的子进程，每隔两秒显示五个页面错误统计信息报告。仅显示具有非零统计值的子进程。 |
| sar -u 2 5                            | Report CPU utilization for each 2 seconds. 5 lines are displayed. | 每2秒报告一次CPU利用率。显示5行。                            |
| sar -I 14 -o int14.file 2 10          | Report statistics on IRQ 14 for each 2 seconds. 10 lines are displayed. Data are stored in a file called int14.file. | 每2秒报告一次IRQ 14统计信息。显示10行。 数据存储在名为int14.file的文件中。 |
| sar -r -n DEV -f /var/log/sa/sa16     | Display memory and network statistics saved in daily data file 'sa16'. | 显示保存在每日数据文件“ sa16”中的内存和网络统计信息。        |
| sar -A                                | Display all the statistics saved in current daily data file. | 显示保存在当前每日数据文件中的所有统计信息。                 |
| sadf -d /var/log/sa/sa21 -- -r -n DEV | Extract memory and network statistics from system activity file 'sa21', and display them in a format that can be ingested by a database. | 从系统活动文件“ sa21”中提取内存和网络统计信息，并以数据库可以提取的格式显示它们。 |
| sadf -p -P 1                          | Extract CPU statistics for processor 1 (the second processor) from current daily data file, and display them in a format that can easily be handled by a pattern processing command. | 从当前的每日数据文件中提取处理器1（第二个处理器）的CPU统计信息，并以可以通过模式处理命令轻松处理的格式显示它们。 |



### 1.4 更多相关

| Name | DESCRIPTION                                                  | 描述                                               |
| ---- | ------------------------------------------------------------ | -------------------------------------------------- |
| sa1  | Collect and store binary data in the system activity daily data file. | 收集二进制数据并将其存储在系统活动每日数据文件中。 |
| sa2  | Create a report from the current standard system activity daily data file. | 根据当前的标准系统活动每日数据文件创建报告。       |
| sadc | System activity data collector.                              | 系统活动数据收集器。                               |
| sadf | Display data collected by sar in multiple formats            | 以多种格式显示sar收集的数据                        |
| sar  | Collect, report, or save system activity information         | 收集，报告或保存系统活动信息                       |

...

| Name     | FILES                                                  | DESCRIPTION                                                  |                                                              |
| -------- | ------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| sa1 sadf | /var/log/sa/saDD /var/log/sa/saYYYYMMDD                | The standard system activity daily data files and their default location. YYYY stands for the current year, MM for the current month and DD for the current day. | 标准系统活动每日数据文件及其默认位置。 YYYY代表当年，MM代表当月，DD代表当日。 |
| sar sadc | /var/log/sa/saDD /var/log/sa/saYYYYMMDD /proc and /sys | The standard system activity daily data files and their default location. YYYY stands for the current year, MM for the current month and DD for the current day. Contain various files with system statistics. | 标准系统活动每日数据文件及其默认位置。 YYYY代表当年，MM代表当月，DD代表当日。 包含各种带有系统统计信息的文件。 |
| sa2      | /var/log/sa/sarDD /var/log/sa/sarYYYYMMDD              | The standard system activity daily report files and their default location. YYYY stands for the current year, MM for the current month and DD for the current day. | 标准系统活动每日报告文件及其默认位置。 YYYY代表当年，MM代表当月，DD代表当日。 |

...

| Name | EXAMPLES                                                     | DESCRIPTION                                                  |                                                              |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| sa1  | 0,10,20,30,40,50 * * * * /usr/lib/sa/sa1 1 1 -S DISK         | To collect data (including those from disks) every 10 minutes, place the following entry in your root crontab file: | 要每10分钟收集一次数据（包括来自磁盘的数据），请将以下条目放入您的crontab根文件中： |
| sa2  | 5 19 * * 1-5 /usr/lib/sa/sa2 -A                              | To run the sa2 command daily, place the following entry in your root crontab file: This will generate by default a daily report called sarDD in the /var/log/sa directory, where the DD parameter is a number representing the day of the month. | 要每天运行sa2命令，请将以下条目放入您的crontab根文件中：默认情况下，它将在/ var / log / sa目录中生成一个名为sarDD的每日报告，其中DD参数是代表月份的数字。 。 |
| sadc | /usr/lib/sa/sadc 1 10 /tmp/datafile /usr/lib/sa/sadc -C Backup_Start /tmp/datafile | Write 10 records of one second intervals to the /tmp/datafile binary file. Insert the comment Backup_Start into the file /tmp/datafile. | 将一秒间隔的10条记录写入/ tmp / datafile二进制文件。将注释Backup_Start插入文件/ tmp / datafile中。 |
| sar  | sar -u 2 5                                                   | Report CPU utilization for each 2 seconds. 5 lines are displayed. | 每2秒报告一次CPU利用率。显示5行。                            |
| sar  | sar -I 14 -o int14.file 2 10                                 | Report statistics on IRQ 14 for each 2 seconds. 10 lines are displayed. Data are stored in a file called int14.file. | 每2秒报告一次IRQ 14统计信息。显示10行。数据存储在名为int14.file的文件中。 |
| sar  | sar -r -n DEV -f /var/log/sa/sa16                            | Display memory and network statistics saved in daily data file 'sa16'. | 显示保存在每日数据文件“ sa16”中的内存和网络统计信息。        |
| sar  | sar -A                                                       | Display all the statistics saved in current daily data file. | 显示保存在当前每日数据文件中的所有统计信息。                 |
| sadf | sadf -d /var/log/sa/sa21 -- -r -n DEV                        | Extract memory and network statistics from system activity file 'sa21', and display them in a format that can be ingested by a database. | 从系统活动文件“ sa21”中提取内存和网络统计信息，并以数据库可以提取的格式显示它们。 |
| sadf | sadf -p -P 1                                                 | Extract CPU statistics for processor 1 (the second processor) from current daily data file, and display them in a format that can easily be handled by a pattern processing command. | 从当前的每日数据文件中提取处理器1（第二个处理器）的CPU统计信息，并以可以通过模式处理命令轻松处理的格式显示它们。 |





















## 2. OPTIONS

### 2.1 man iostat

| OPTIONS                                                      | DESCRIPTION                                                  | cn                                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| -c                                                           | Display the CPU utilization report.                          | 显示CPU利用率报告。                                          |
| **-d**                                                       | Display the device utilization report.                       | 显示设备利用率报告。                                         |
| --dec={ 0 \| 1 \| 2 }                                        | Specify the number of decimal places to use (0 to 2, default value is 2). | 指定要使用的小数位数（0到2，默认值为2）。                    |
| -g group_name { device [...] \| ALL }                        | Display statistics for a group of devices. The iostat command reports statistics for each individual device in the list then a line of global statistics for the group displayed as group_name and made up of all the devices in the list. The ALL keyword means that all the block devices defined by the system shall be included in the group. | 显示一组设备的统计信息。 iostat命令报告列表中每个单独设备的统计信息，然后报告显示为group_name并由列表中所有设备组成的组的全局统计信息行。 ALL关键字表示系统定义的所有块设备都应包括在该组中。 |
| -H                                                           | This option must be used with option -g and indicates that only global statistics for the group are to be displayed, and not statistics for individual devices in the group. | 此选项必须与选项-g一起使用，并指示仅显示该组的全局统计信息，而不显示该组中各个设备的统计信息。 |
| -h                                                           | Make the Device Utilization Report easier to read by a human. --human is enabled implicitly with this option. | 使设备使用情况报告更易于人阅读。 --human使用此选项隐式启用。 |
| --human                                                      | Print sizes in human readable format (e.g. 1.0k, 1.2M, etc.) The units displayed with this option supersede any other default units (e.g. kilobytes, sectors...) associated with the metrics. | 以人类可读格式打印的尺寸（例如1.0k，1.2M等）。使用此选项显示的单位将取代与度量标准关联的任何其他默认单位（例如千字节，扇区...）。 |
| -j { ID \| LABEL \| PATH \| UUID \| ... } [ device [...] \| ALL ] | Display persistent device names. Options ID, LABEL, etc. specify the type of the persistent name. These options are not limited, only prerequisite is that directory with required persistent names is present in /dev/disk. Optionally, multiple devices can be specified in the chosen persistent name type. Because persistent device names are usually long, option | 显示永久设备名称。选项ID，LABEL等指定持久名称的类型。这些选项不受限制，只有先决条件是/ dev / disk中存在具有所需持久名称的目录。 （可选）可以在所选的持久名称类型中指定多个设备。因为持久性设备名称通常很长，所以选择 |
| -k                                                           | Display statistics in kilobytes per second.                  | 显示统计信息（以千字节/秒为单位）。                          |
| -m                                                           | Display statistics in megabytes per second.                  | 以每秒兆字节显示统计信息。                                   |
| -N                                                           | Display the registered device mapper names for any device mapper devices. Useful for viewing LVM2 statistics. | 显示任何设备映射器设备的注册设备映射器名称。用于查看LVM2统计信息。 |
| -o JSON                                                      | Display the statistics in JSON (Javascript Object Notation) format. JSON output field order is undefined, and new fields may be added in the future. | 以JSON（JavaScript对象表示法）格式显示统计信息。 JSON输出字段顺序未定义，将来可能会添加新字段。 |
| **-p [ {device [,...] \| ALL } ]**                           | The -p option displays statistics for block devices and all their partitions that are used by the system. If a device name is entered on the command line, then statistics for it and all its partitions are displayed. Last, the ALL keyword indicates that statistics have to be displayed for all the block devices and partitions defined by the system, including those that have never been used. If option -j is defined before this option, devices entered on the command line can be specified with the chosen persistent name type. | -p选项显示系统使用的块设备及其所有分区的统计信息。如果在命令行上输入了设备名称，则将显示该设备及其所有分区的统计信息。最后，ALL关键字指示必须显示系统定义的所有块设备和分区的统计信息，包括从未使用过的统计信息。如果在此选项之前定义了选项-j，则可以使用所选的持久名称类型指定在命令行上输入的设备。 |
| -s                                                           | Display a short (narrow) version of the report that should fit in 80 characters wide screens. | 在80个字符的宽屏幕中显示报告的简短（窄版）版本。             |
| -t                                                           | Print the time for each report displayed. The timestamp format may depend on the value of the S_TIME_FORMAT environment variable (see below). | 打印显示的每个报告的时间。时间戳格式可能取决于S_TIME_FORMAT环境变量的值（请参见下文）。 |
| -V                                                           | Print version number then exit.                              | 打印版本号，然后退出。                                       |
| **-x**                                                       | Display extended statistics.                                 | 显示扩展统计信息。                                           |
| -y                                                           | Omit first report with statistics since system boot, if displaying multiple records at given interval. | 如果以给定的时间间隔显示多个记录，则自系统启动以来忽略带有统计信息的第一个报告。 |
| -z                                                           | Tell iostat to omit output for any devices for which there was no activity during the sample period. | 告诉iostat忽略在采样期间没有任何活动的任何设备的输出。       |

### 2.2 man mpstat

| OPTIONS                      | DESCRIPTION                                                  | cn                                                           |
| ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| -A                           | This option is equivalent to specifying -n -u -I ALL. This option also implies specifying -N ALL -P ALL unless these options are explicitly set on the command line. | 此选项等效于指定-n -u -I ALL。该选项还意味着指定-N ALL -P ALL，除非在命令行上显式设置了这些选项。 |
| --dec={ 0 \| 1 \| 2 }        | Specify the number of decimal places to use (0 to 2, default value is 2). | 指定要使用的小数位数（0到2，默认值为2）。                    |
| -I { keyword [,...] \| ALL } | Report interrupts statistics. Possible keywords are CPU, SCPU, and SUM. With the CPU keyword, the number of each individual interrupt received per second by the CPU or CPUs is displayed. Interrupts are those listed in /proc/interrupts file. With the SCPU keyword, the number of each individual software interrupt received per second by the CPU or CPUs is displayed. This option works only with kernels 2.6.31 and later. Software interrupts are those listed in /proc/softirqs file. With the SUM keyword, the mpstat command reports the total number of interrupts per processor. The following values are displayed: | 报告中断统计信息。可能的关键字是CPU，SCPU和SUM。使用CPU关键字，显示一个或多个CPU每秒接收到的每个单独中断的数量。中断是/ proc / interrupts文件中列出的中断。使用SCPU关键字，显示一个或多个CPU每秒接收的每个软件中断的数量。此选项仅适用于内核2.6.31及更高版本。软件中断是/ proc / softirqs文件中列出的中断。使用SUM关键字，mpstat命令报告每个处理器的中断总数。显示以下值： |
| CPU                          | Processor number. The keyword all indicates that statistics are calculated as averages among all processors. | 处理器编号。关键字all表示统计信息是所有处理器之间的平均值。  |
| intr/s                       | Show the total number of interrupts received per second by the CPU or CPUs. | 显示一个或多个CPU每秒接收的中断总数。                        |
|                              | The ALL keyword is equivalent to specifying all the keywords above and therefore all the interrupts statistics are displayed. | ALL关键字等效于指定上面的所有关键字，因此将显示所有中断统计信息。 |
| -N { node_list \| ALL }      | Indicate the NUMA nodes for which statistics are to be reported. node_list is a list of comma-separated values or range of values (e.g., 0,2,4-7,12-). Note that node all is the global average among all nodes. The ALL keyword indicates that statistics are to be reported for all nodes. | 指示要报告其统计信息的NUMA节点。 node_list是逗号分隔值或值范围（例如0、2、4-7、12-）的列表。注意，所有节点是所有节点之间的全局平均值。 ALL关键字指示要报告所有节点的统计信息。 |
| -n                           | Report summary CPU statistics based on NUMA node placement. The following values are displayed: | 报告基于NUMA节点位置的摘要CPU统计信息。显示以下值：          |
| NODE                         | Logical NUMA node number. The keyword all indicates that statistics are calculated as averages among all nodes. | 逻辑NUMA节点号。关键字all表示统计量是所有节点之间的平均值。  |
|                              | All the other fields are the same as those displayed with option -u (see below). | 其他所有字段与使用-u选项显示的字段相同（请参见下文）。       |
| -o JSON                      | Display the statistics in JSON (Javascript Object Notation) format. JSON output field order is undefined, and new fields may be added in the future. | 以JSON（JavaScript对象表示法）格式显示统计信息。 JSON输出字段顺序未定义，将来可能会添加新字段。 |
| **-P { cpu_list \| ALL }**   | Indicate the processors for which statistics are to be reported. cpu_list is a list of comma-separated values or range of values (e.g., 0,2,4-7,12-). Note that processor 0 is the first processor, and processor all is the global average among all processors. The ALL keyword indicates that statistics are to be reported for all processors. Offline processors are not displayed. | 指示要报告其统计信息的处理器。 cpu_list是逗号分隔值或值范围（例如0、2、4-7、12-）的列表。请注意，处理器0是第一个处理器，所有处理器是所有处理器之间的全局平均值。 ALL关键字指示要报告所有处理器的统计信息。不显示脱机处理器。 |
| -T                           | Display topology elements in the CPU report (see option -u below). The following elements are displayed: | 在CPU报告中显示拓扑元素（请参阅下面的选项-u）。显示以下元素： |
| CORE                         | Logical core number.                                         | 逻辑核心号。                                                 |
| SOCK                         | Logical socket number.                                       | 逻辑套接字号。                                               |
| NODE                         | Logical NUMA node number.                                    | 逻辑NUMA节点号。                                             |
| -u                           | Report CPU utilization. The following values are displayed:  | 报告CPU利用率。显示以下值：                                  |
| CPU                          | Processor number. The keyword all indicates that statistics are calculated as averages among all processors. | 处理器编号。关键字all表示统计信息是所有处理器之间的平均值。  |
| %usr                         | Show the percentage of CPU utilization that occurred while executing at the user level (application). | 显示在用户级别（应用程序）执行时发生的CPU利用率百分比。      |
| %nice                        | Show the percentage of CPU utilization that occurred while executing at the user level with nice priority. | 显示在用户级别执行时具有较高优先级的CPU利用率百分比。        |
| %sys                         | Show the percentage of CPU utilization that occurred while executing at the system level (kernel). Note that this does not include time spent servicing hardware and software interrupts. | 显示在系统级别（内核）执行时发生的CPU利用率百分比。请注意，这不包括花在维修硬件和软件中断上的时间。 |
| %iowait                      | Show the percentage of time that the CPU or CPUs were idle during which the system had an outstanding disk I/O request. | 显示在系统有未完成的磁盘I / O请求期间，一个或多个CPU空闲的时间百分比。 |
| %irq                         | Show the percentage of time spent by the CPU or CPUs to service hardware interrupts. | 显示一个或多个CPU服务硬件中断所花费的时间百分比。            |
| %soft                        | Show the percentage of time spent by the CPU or CPUs to service software interrupts. | 显示一个或多个CPU服务软件中断所花费的时间百分比。            |
| %steal                       | Show the percentage of time spent in involuntary wait by the virtual CPU or CPUs while the hypervisor was servicing another virtual processor. | 显示在管理程序为另一个虚拟处理器提供服务时，一个或多个虚拟CPU在非自愿等待中花费的时间百分比。 |
| %guest                       | Show the percentage of time spent by the CPU or CPUs to run a virtual processor. | 显示一个或多个CPU运行虚拟处理器所花费的时间百分比。          |
| %gnice                       | Show the percentage of time spent by the CPU or CPUs to run a niced guest. | 显示一个或多个CPU运行一个好的guest虚拟机所花费的时间百分比。 |
| %idle                        | Show the percentage of time that the CPU or CPUs were idle and the system did not have an outstanding disk I/O request. | 显示一个或多个CPU空闲且系统没有未完成的磁盘I / O请求的时间百分比。 |
| -V                           | Print version number then exit.                              | 打印版本号，然后退出。                                       |

### 2.3 man pidstat

| OPTIONS                              | DESCRIPTION                                                  | cn                                                           |
| ------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **-C comm**                          | Display only tasks whose command name includes the string comm. This string can be a regular expression. | 仅显示命令名称包含字符串comm的任务。该字符串可以是正则表达式。 |
| **-d**                               | Report I/O statistics (kernels 2.6.20 and later only). The following values may be displayed: | 报告I / O统计信息（仅内核2.6.20及更高版本）。可能显示以下值： |
| UID                                  | The real user identification number of the task being monitored. | 被监视任务的真实用户标识号。                                 |
| USER                                 | The name of the real user owning the task being monitored.   | 拥有要监视的任务的真实用户的名称。                           |
| PID                                  | The identification number of the task being monitored.       | 被监视任务的标识号。                                         |
| kB_rd/s                              | Number of kilobytes the task has caused to be read from disk per second. | 每秒导致从磁盘读取任务的千字节数。                           |
| kB_wr/s                              | Number of kilobytes the task has caused, or shall cause to be written to disk per second. | 任务已导致或应导致每秒将其写入磁盘的千字节数。               |
| kB_ccwr/s                            | Number of kilobytes whose writing to disk has been cancelled by the task. This may occur when the task truncates some dirty pagecache. In this case, some IO which another task has been accounted for will not be happening. | 任务已取消其写入磁盘的千字节数。当任务截断一些脏页缓存时，可能会发生这种情况。在这种情况下，已经解决了另一个任务的某些IO将不会发生。 |
| iodelay                              | Block I/O delay of the task being monitored, measured in clock ticks. This metric includes the delays spent waiting for sync block I/O completion and for swapin block I/O completion. | 被监视任务的块I / O延迟，以时钟周期为单位。此度量标准包括等待同步块I / O完成和交换模块I / O完成所花费的延迟。 |
| Command                              | The command name of the task.                                | 任务的命令名称。                                             |
| --dec={ 0 \| 1 \| 2 }                | Specify the number of decimal places to use (0 to 2, default value is 2). | 指定要使用的小数位数（0到2，默认值为2）。                    |
| -e program args                      | Execute program with given arguments args and monitor it with pidstat. pidstat stops when program terminates. | 使用给定参数args执行程序，并使用pidstat对其进行监视。程序终止时，pidstat停止。 |
| -G process_name                      | Display only processes whose command name includes the string process_name. This string can be a regular expression. If option -t is used together with option -G then the threads belonging to that process are also displayed (even if their command name doesn't include the string process_name). | 仅显示命令名称包含字符串process_name的进程。该字符串可以是正则表达式。如果选项-t与选项-G一起使用，则还将显示属于该进程的线程（即使它们的命令名不包含字符串process_name）。 |
| -H                                   | Display timestamp in seconds since the epoch.                | 以秒为单位显示时间戳记。                                     |
| -h                                   | Display all activities horizontally on a single line, with no average statistics at the end of the report. This is intended to make it easier to be parsed by other programs. | 在一行上水平显示所有活动，报告末尾没有平均统计信息。目的是使它易于被其他程序解析。 |
| --human                              | Print sizes in human readable format (e.g. 1.0k, 1.2M, etc.) The units displayed with this option supersede any other default units (e.g. kilobytes, sectors...) associated with the metrics. | 以人类可读格式打印的尺寸（例如1.0k，1.2M等）。使用此选项显示的单位将取代与度量标准关联的任何其他默认单位（例如千字节，扇区...）。 |
| -I                                   | In an SMP environment, indicate that tasks CPU usage (as displayed by option -u ) should be divided by the total number of processors. | 在SMP环境中，指示应将任务CPU使用率（如选项-u所示）除以处理器总数。 |
| -l                                   | Display the process command name and all its arguments.      | 显示流程命令名称及其所有参数。                               |
| **-p { pid [,...] \| SELF \| ALL }** | Select tasks (processes) for which statistics are to be reported. pid is the process identification number. The SELF keyword indicates that statistics are to be reported for the pidstat process itself, whereas the ALL keyword indicates that statistics are to be reported for all the tasks managed by the system. | 选择要为其报告统计信息的任务（过程）。 pid是进程标识号。 SELF关键字指示要为pidstat进程本身报告统计信息，而ALL关键字指示要为系统管理的所有任务报告统计信息。 |
| -R                                   | Report realtime priority and scheduling policy information. The following values may be displayed: | 报告实时优先级和调度策略信息。可能显示以下值：               |
| UID, USER, PID, Command              |                                                              |                                                              |
| prio                                 | The realtime priority of the task being monitored.           | 被监视任务的实时优先级。                                     |
| policy                               | The scheduling policy of the task being monitored.           | 被监视任务的调度策略。                                       |
| **-r**                               | Report page faults and memory utilization. When reporting statistics for individual tasks, the following values may be displayed: | 报告页面错误和内存利用率。当报告单个任务的统计信息时，可能会显示以下值： |
| UID, USER, PID, Command              |                                                              |                                                              |
| minflt/s                             | Total number of minor faults the task has made per second, those which have not required loading a memory page from disk. | 每秒任务执行的次要故障总数，即不需要从磁盘加载内存页面的次要故障总数。 |
| majflt/s                             | Total number of major faults the task has made per second, those which have required loading a memory page from disk. | 每秒任务执行的主要故障总数，即需要从磁盘加载内存页面的主要故障总数。 |
| VSZ                                  | Virtual Size: The virtual memory usage of entire task in kilobytes. | 虚拟大小：整个任务的虚拟内存使用量（以千字节为单位）。       |
| RSS                                  | Resident Set Size: The non-swapped physical memory used by the task in kilobytes. | 驻留集大小：任务使用的未交换的物理内存，以千字节为单位。     |
| %MEM                                 | The tasks's currently used share of available physical memory. | 任务当前使用的可用物理内存份额。                             |
|                                      | When reporting global statistics for tasks and all their children, the following values may be displayed: | 当报告任务及其所有子任务的全局统计信息时，可能会显示以下值： |
| UID                                  | The real user identification number of the task which is being monitored together with its children. | 与其子项一起监视的任务的真实用户标识号。                     |
| USER                                 | The name of the real user owning the task which is being monitored together with its children. | 拥有正在被监视的任务及其子项的真实用户的名称。               |
| PID                                  | The identification number of the task which is being monitored together with its children. | 与其子项一起监视的任务的标识号。                             |
| minflt-nr                            | Total number of minor faults made by the task and all its children, and collected during the interval of time. | 任务及其所有子项所犯的并在一段时间间隔内收集的小错误总数。   |
| majflt-nr                            | Total number of major faults made by the task and all its children, and collected during the interval of time. | 任务及其所有子项所造成并在时间间隔内收集的主要故障总数。     |
| Command                              | The command name of the task which is being monitored together with its children. | 与其子项一起监视的任务的命令名称。                           |
| -s                                   | Report stack utilization. The following values may be displayed: | 报告堆栈利用率。可能显示以下值：                             |
| UID, USER, PID, Command              |                                                              |                                                              |
| StkSize                              | The amount of memory in kilobytes reserved for the task as stack, but not necessarily used. | 为任务保留为堆栈的内存量（以千字节为单位），但不一定使用。   |
| StkRef                               | The amount of memory in kilobytes used as stack, referenced by the task. | 任务引用的用作堆栈的内存量（以千字节为单位）。               |
| **-T { TASK \| CHILD \| ALL }**      | This option specifies what has to be monitored by the pidstat command. The TASK keyword indicates that statistics are to be reported for individual tasks (this is the default option) whereas the CHILD keyword indicates that statistics are to be globally reported for the selected tasks and all their children. The ALL keyword indicates that statistics are to be reported for individual tasks and globally for the selected tasks and their children. Note: Global statistics for tasks and all their children are not available for all options of pidstat. Also these statistics are not necessarily relevant to current time interval: The statistics of a child process are collected only when it finishes or it is killed. | 此选项指定pidstat命令必须监视的内容。 TASK关键字指示要报告单个任务的统计信息（这是默认选项），而CHILD关键字指示要针对所选任务及其所有子项全局报告统计信息。 ALL关键字指示要报告单个任务的统计信息，并针对所选任务及其子项全局报告统计信息。注意：任务及其所有子项的全局统计信息不适用于pidstat的所有选项。同样，这些统计信息不一定与当前时间间隔相关：子进程的统计信息仅在子进程完成或被杀死时才收集。 |
| -t                                   | Also display statistics for threads associated with selected tasks. This option adds the following values to the reports: | 还显示与所选任务关联的线程的统计信息。此选项将以下值添加到报告中： |
| TGID                                 | The identification number of the thread group leader.        | 线程组负责人的标识号。                                       |
| TID                                  | The identification number of the thread being monitored.     | 被监视线程的标识号。                                         |
| -U [ username ]                      | Display the real user name of the tasks being monitored instead of the UID. If username is specified, then only tasks belonging to the specified user are displayed. | 显示要监视的任务的真实用户名，而不是UID。如果指定了username，则仅显示属于指定用户的任务。 |
| **-u**                               | Report CPU utilization. When reporting statistics for individual tasks, the following values may be displayed: | 报告CPU利用率。当报告单个任务的统计信息时，可能会显示以下值： |
| UID, USER, PID, Command              |                                                              |                                                              |
| %usr                                 | Percentage of CPU used by the task while executing at the user level (application), with or without nice priority. Note that this field does NOT include time spent running a virtual processor. | 在用户级别（应用程序）执行时，任务具有优先级或没有优先级的任务所使用的CPU的百分比。请注意，此字段不包括运行虚拟处理器所花费的时间。 |
| %system                              | Percentage of CPU used by the task while executing at the system level (kernel). | 在系统级别（内核）执行时任务使用的CPU百分比。                |
| %guest                               | Percentage of CPU spent by the task in virtual machine (running a virtual processor). | 任务在虚拟机（运行虚拟处理器）中花费的CPU百分比。            |
| %wait                                | Percentage of CPU spent by the task while waiting to run.    | 等待运行时任务花费的CPU百分比。                              |
| %CPU                                 | Total percentage of CPU time used by the task. In an SMP environment, the task's CPU usage will be divided by the total number of CPU's if option -I has been entered on the command line. | 任务使用的CPU时间的总百分比。在SMP环境中，如果在命令行中输入选项-I，则任务的CPU使用率将除以CPU的总数。 |
| CPU                                  | Processor number to which the task is attached.              | 任务附加到的处理器号。                                       |
|                                      | When reporting global statistics for tasks and all their children, the following values may be displayed: | 当报告任务及其所有子任务的全局统计信息时，可能会显示以下值： |
| UID, USER, PID, Command              |                                                              |                                                              |
| usr-ms                               | Total number of milliseconds spent by the task and all its children while executing at the user level (application), with or without nice priority, and collected during the interval of time. Note that this field does NOT include time spent running a virtual processor. | 在用户级别（应用程序）执行任务时，任务及其所有子级花费的毫秒总数（有或没有很好的优先级），并在时间间隔内收集。请注意，此字段不包括运行虚拟处理器所花费的时间。 |
| system-ms                            | Total number of milliseconds spent by the task and all its children while executing at the system level (kernel), and collected during the interval of time. | 在系统级别（内核）执行时，任务及其所有子代花费的总毫秒数，该时间间隔是在该时间间隔内收集的。 |
| guest-ms                             | Total number of milliseconds spent by the task and all its children in virtual machine (running a virtual processor). | 任务及其所有子代在虚拟机（运行虚拟处理器）中花费的总毫秒数。 |
| -V                                   | Print version number then exit.                              | 打印版本号，然后退出。                                       |
| -v                                   | Report values of some kernel tables. The following values may be displayed: | 报告某些内核表的值。可能显示以下值：                         |
| UID, USER, PID, Command              |                                                              |                                                              |
| threads                              | Number of threads associated with current task.              | 与当前任务关联的线程数。                                     |
| fd-nr                                | Number of file descriptors associated with current task.     | 与当前任务关联的文件描述符数。                               |
| **-w**                               | Report task switching activity (kernels 2.6.23 and later only). The following values may be displayed: | 报告任务切换活动（仅内核2.6.23及更高版本）。可能显示以下值： |
| UID, USER, PID, Command              |                                                              |                                                              |
| cswch/s                              | Total number of voluntary context switches the task made per second. A voluntary context switch occurs when a task blocks because it requires a resource that is unavailable. | 每秒执行的任务的自愿上下文切换总数。当任务因其需要的资源不可用而被阻止时，会发生自愿上下文切换。 |
| nvcswch/s                            | Total number of non voluntary context switches the task made per second. A involuntary context switch takes place when a task executes for the duration of its time slice and then is forced to relinquish the processor. | 每秒执行的非自愿上下文切换总数。当任务在其时间片的持续时间内执行，然后被迫放弃处理器时，会发生非自愿上下文切换。 |

### 2.4 man sadf

| OPTIONS                    | DESCRIPTION                                                  | cn                                                           |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| -C                         | Tell sadf to display comments present in file.               | 告诉sadf显示文件中存在的注释。                               |
| -c                         | Convert an old system activity binary datafile (version 9.1.6 and later) to current up-to-date format. Use the following syntax: sadf -c old_datafile > new_datafile Conversion can be controlled using option -O (see below). | 将旧的系统活动二进制数据文件（版本9.1.6及更高版本）转换为当前最新格式。使用以下语法：sadf -c old_datafile> new_datafile可以使用-O选项控制转换（请参见下文）。 |
| **-d**                     | Print the contents of the data file in a format that can easily be ingested by a relational database system. The output consists of fields separated by a semicolon. Each record contains the hostname of the host where the file was created, the interval value (or -1 if not applicable), the timestamp in a form easily acceptable by most databases, and additional semicolon separated data fields as specified by sar_options command line options. Note that timestamp output can be controlled by options -T, -t and -U. | 以一种关系数据库系统可以轻松摄取的格式打印数据文件的内容。输出由用分号分隔的字段组成。每个记录均包含创建文件的主机的主机名，间隔值（或-1，如果不适用），大多数数据库容易接受的格式的时间戳以及由sar_options命令行选项指定的其他用分号分隔的数据字段。请注意，时间戳输出可以通过选项-T，-t和-U控制。 |
| --dev=dev_list             | Specify the block devices for which statistics are to be displayed by sadf. dev_list is a list of comma-separated device names. Useful with option -d from sar. | 指定sadf要显示其统计信息的块设备。 dev_list是用逗号分隔的设备名称的列表。与sar的-d选项一起使用。 |
| -e [ hh:mm[:ss] ]          | Set the ending time of the report. The default ending time is 18:00:00. Hours must be given in 24-hour format. | 设置报告的结束时间。默认结束时间为18:00:00。小时必须以24小时格式给出。 |
| --fs=fs_list               | Specify the filesystems for which statistics are to be displayed by sadf. fs_list is a list of comma-separated filesystem names or mountpoints. Useful with option -F from sar. | 指定sadf要显示其统计信息的文件系统。 fs_list是用逗号分隔的文件系统名称或挂载点的列表。与sar的-F选项一起使用。 |
| -g                         | Print the contents of the data file in SVG (Scalable Vector Graphics) format. This option enables you to display some fancy graphs in your web browser. Use the following syntax: sadf -g your_datafile [ -- sar_options ] > output.svg and open the resulting SVG file in your favorite web browser. Output can be controlled using option -O (see below). | 以SVG（可缩放矢量图形）格式打印数据文件的内容。此选项使您可以在Web浏览器中显示一些奇特的图形。使用以下语法：sadf -g your_datafile [-sar_options]> output.svg，然后在您喜欢的Web浏览器中打开生成的SVG文件。可以使用-O选项控制输出（请参见下文）。 |
| -H                         | Display only the header of the report (when applicable). If no format has been specified, then the header data (metadata) of the data file are displayed. | 仅显示报告的标题（如果适用）。如果未指定格式，则显示数据文件的头数据（元数据）。 |
| -h                         | When used in conjunction with option -d, all activities will be displayed horizontally on a single line. | 与选项-d结合使用时，所有活动将水平显示在一行上。             |
| --iface=iface_list         | Specify the network interfaces for which statistics are to be displayed by sadf. iface_list is a list of comma-separated interface names. Useful with options -n DEV and -n EDEV from sar. | 指定Sadf要显示其统计信息的网络接口。 iface_list是用逗号分隔的接口名称的列表。可用于sar的-n DEV和-n EDEV选项。 |
| -j                         | Print the contents of the data file in JSON (JavaScript Object Notation) format. Timestamps can be controlled by options -T and -t. | 以JSON（JavaScript对象表示法）格式打印数据文件的内容。时间戳记可以通过选项-T和-t来控制。 |
| -l                         | Export the contents of the data file to a PCP (Performance Co-Pilot) archive. The name of the archive can be specified using the keyword pcparchive= with option -O. | 将数据文件的内容导出到PCP（Performance Co-Pilot）归档中。可以使用带有选项-O的关键字pcparchive =来指定档案的名称。 |
| -O opts [,…]               | Use the specified options to control the output of sadf. The following options are used to control SVG output displayed by sadf -g: | 使用指定的选项来控制sadf的输出。以下选项用于控制sadf -g显示的SVG输出： |
| autoscale                  | Draw all the graphs of a given view as large as possible based on current view's scale. To do this, a factor (10, 100, 1000...) is used to enlarge the graph drawing. This option may be interesting when several graphs are drawn on the same view, some with only very small values, and others with high ones, the latter making the former hardly visible. | 根据当前视图的比例绘制尽可能大的给定视图的所有图形。为此，使用系数（10，100，1000 ...）来放大图形。当在同一视图上绘制多个图形时，此选项可能很有趣，有些图形的值很小，而有些图形的值很高，而后者使前者几乎不可见。 |
| bwcol                      | Use a black and white palette to draw the graphs.            | 使用黑白调色板绘制图形。                                     |
| customcol                  | Use a customizable color palette instead of the default one to draw the graphs. See environment variable S_COLORS_PALETTE below to know how to customize that palette. | 使用可自定义的调色板而不是默认调色板来绘制图形。请参阅下面的环境变量S_COLORS_PALETTE以了解如何自定义该调色板。 |
| height=value               | Set SVG canvas height to value.                              | 将SVG画布高度设置为value。                                   |
| oneday                     | Display graphs data over a period of 24 hours. Note that hours are still printed in UTC by default: You should use option -T to print them in local time and get a time window starting from midnight. | 在24小时内显示图形数据。请注意，默认情况下，小时仍以UTC进行打印：您应使用选项-T在当地时间进行打印，并获得从午夜开始的时间窗口。 |
| packed                     | Group all views from the same activity (and for the same device) on the same row. | 将来自同一活动（针对同一设备）的所有视图归为同一行。         |
| showidle                   | Also display %idle state in graphs for CPU statistics.       | 还可以在图形中显示％idle状态以获取CPU统计信息。              |
| showinfo                   | Display additional information (such as the date and the host name) on each view. | 在每个视图上显示其他信息（例如日期和主机名）。               |
| showtoc                    | Add a table of contents at the beginning of the SVG output, consisting of links pointing at the first graph of each activity. | 在SVG输出的开头添加一个目录，该目录由指向每个活动的第一个图的链接组成。 |
| skipempty                  | Do not display views where all graphs have only zero values. | 不要显示所有图形都只有零值的视图。                           |
|                            | The following option may be used when converting an old system activity binary datafile to current up-to-date format: | 将旧的系统活动二进制数据文件转换为当前最新格式时，可以使用以下选项： |
| hz=value                   | Specify the number of ticks per second for the machine where the old datafile has been created. | 为创建旧数据文件的机器指定每秒的滴答数。                     |
|                            | The following option may be used when data are exported to a PCP archive: | 将数据导出到PCP存档时，可以使用以下选项：                    |
| pcparchive=name            | Specify the name of the PCP archive to create.               | 指定要创建的PCP存档的名称。                                  |
|                            | The following option is used to control raw output displayed by sadf -r: | 以下选项用于控制sadf -r显示的原始输出：                      |
| debug                      | Display additional information, mainly useful for debugging purpose. | 显示其他信息，主要用于调试目的。                             |
| **-P { cpu_list \| ALL }** | Tell sadf that processor dependent statistics are to be reported only for the specified processor or processors. cpu_list is a list of comma-separated values or range of values (e.g., 0,2,4-7,12-). Note that processor 0 is the first processor, and processor all is the global average among all processors. Specifying the ALL keyword reports statistics for each individual processor, and globally for all processors. | 告诉sadf仅依赖于一个或多个指定处理器的处理器相关统计信息将被报告。 cpu_list是逗号分隔值或值范围（例如0、2、4-7、12-）的列表。请注意，处理器0是第一个处理器，所有处理器是所有处理器之间的全局平均值。指定ALL关键字报告每个处理器的统计信息，并全局报告所有处理器的统计信息。 |
| **-p**                     | Print the contents of the data file in a format that can easily be handled by pattern processing commands like awk. The output consists of fields separated by a tab. Each record contains the hostname of the host where the file was created, the interval value (or -1 if not applicable), the timestamp, the device name (or - if not applicable), the field name and its value. Note that timestamp output can be controlled by options -T, -t and -U. | 以可以通过模式处理命令（如awk）轻松处理的格式打印数据文件的内容。输出包含由制表符分隔的字段。每个记录都包含创建文件所在主机的主机名，间隔值（如果不适用，则为-1），时间戳，设备名称（或-如果不适用），字段名称及其值。请注意，时间戳输出可以通过选项-T，-t和-U控制。 |
| **-r**                     | Print the raw contents of the data file. With this format, the values for all the counters are displayed as read from the kernel, which means e.g., that no average values are calculated over the elapsed time interval. Output can be controlled using option -O (see above). | 打印数据文件的原始内容。使用这种格式，所有计数器的值都显示为从内核读取的值，这意味着，例如，在经过的时间间隔内未计算平均值。可以使用-O选项控制输出（请参见上文）。 |
| -s [ hh:mm[:ss] ]          | Set the starting time of the data, causing the sadf command to extract records time-tagged at, or following, the time specified. The default starting time is 08:00:00. Hours must be given in 24-hour format. | 设置数据的开始时间，使sadf命令提取指定时间或之后指定时间标记的记录。默认开始时间为08:00:00。小时必须以24小时格式给出。 |
| -T                         | Display timestamp in local time instead of UTC (Coordinated Universal Time). | 以当地时间而不是UTC（世界标准时间）显示时间戳。              |
| -t                         | Display timestamp in the original local time of the data file creator instead of UTC (Coordinated Universal Time). | 在数据文件创建者的原始本地时间而不是UTC（世界标准时间）显示时间戳。 |
| -U                         | Display timestamp (UTC - Coordinated Universal Time) in seconds from the epoch. | 以秒为单位显示时间戳（UTC-协调世界时）。                     |
| -V                         | Print version number then exit.                              | 打印版本号，然后退出。                                       |
| -x                         | Print the contents of the data file in XML format. Timestamps can be controlled by options -T and -t. The corresponding DTD (Document Type Definition) and XML Schema are included in the sysstat source package. They are also available at http://pagesperso-orange.fr/sebastien.godard/download.html | 以XML格式打印数据文件的内容。时间戳记可以通过选项-T和-t来控制。相应的DTD（文档类型定义）和XML模式包含在sysstat源程序包中。也可以在http://pagesperso-orange.fr/sebastien.godard/download.html上找到它们。 |

### 2.5 sar --help

注: man sar 有更加详细的内容, 通常--help里会显示比较常用的选项.

| $ sar --help                          | Usage: sar [ options ] [ <interval> [ <count> ] ]            | Des…                                               |                                     |
| ------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------- | ----------------------------------- |
|                                       | Main options and reports (report name between square brackets): | 主要选项和报告（方括号之间的报告名称）：           |                                     |
| -B                                    | Paging statistics [A_PAGE]                                   | 分页统计信息[A_PAGE]                               |                                     |
| -b                                    | I/O and transfer rate statistics [A_IO]                      | I / O和传输速率统计信息[A_IO]                      |                                     |
| -d                                    | Block devices statistics [A_DISK]                            | 块设备统计信息[A_DISK]                             |                                     |
| -F [ MOUNT ]                          | Filesystems statistics [A_FS]                                | 文件系统统计信息[A_FS]                             |                                     |
| -H                                    | Hugepages utilization statistics [A_HUGE]                    | 大页面利用率统计信息[A_HUGE]                       |                                     |
| **-I { <int_list> \| SUM \| ALL }**   | Interrupts statistics [A_IRQ]                                | 中断统计[A_IRQ]                                    |                                     |
| **-r [ ALL ]**                        | Memory utilization statistics [A_MEMORY]                     | 内存利用率统计信息[A_MEMORY]                       |                                     |
| -S                                    | Swap space utilization statistics [A_MEMORY]                 | 交换空间利用率统计信息[A_MEMORY]                   |                                     |
| **-u [ ALL ]**                        | CPU utilization statistics [A_CPU]                           | CPU利用率统计信息[A_CPU]                           |                                     |
| -v                                    | Kernel tables statistics [A_KTABLES]                         | 内核表统计信息[A_KTABLES]                          |                                     |
| -W                                    | Swapping statistics [A_SWAP]                                 | 交换统计信息[A_SWAP]                               |                                     |
| -w                                    | Task creation and system switching statistics [A_PCSW]       | 任务创建和系统切换统计信息[A_PCSW]                 |                                     |
| -y                                    | TTY devices statistics [A_SERIAL]                            | TTY设备统计信息[A_SERIAL]                          |                                     |
| -m { <keyword> [,...] \| ALL }        | Power management statistics [A_PWR_...].                     | 电源管理统计信息[A_PWR _...]。                     |                                     |
| Keywords are:                         | CPU                                                          | CPU instantaneous clock frequency                  | CPU瞬时时钟频率                     |
|                                       | FAN                                                          | Fans speed                                         | 风扇转速                            |
|                                       | FREQ                                                         | CPU average clock frequency                        | CPU平均时钟频率                     |
|                                       | IN                                                           | Voltage inputs                                     | 电压输入                            |
|                                       | TEMP                                                         | Devices temperature                                | 设备温度                            |
|                                       | USB                                                          | USB devices plugged into the system                | 将USB设备插入系统                   |
| -q [ <keyword> [,...] \| PSI \| ALL ] | System load and pressure-stall statistics.                   | 系统负载和压力失速统计。                           |                                     |
| Keywords are:                         | LOAD                                                         | Queue length and load average statistics [A_QUEUE] | 队列长度和平均负载统计信息[A_QUEUE] |
|                                       | CPU                                                          | Pressure-stall CPU statistics [A_PSI_CPU]          | 失速CPU统计信息[A_PSI_CPU]          |
|                                       | IO                                                           | Pressure-stall I/O statistics [A_PSI_IO]           | 压力失速I / O统计信息[A_PSI_IO]     |
|                                       | MEM                                                          | Pressure-stall memory statistics [A_PSI_MEM]       | 失速内存统计信息[A_PSI_MEM]         |
| **-n { <keyword> [,...] \| ALL }**    | Network statistics [A_NET_…].                                | 网络统计信息[A_NET_…]。                            |                                     |
| Keywords are:                         | DEV                                                          | Network interfaces                                 |                                     |
|                                       | EDEV                                                         | Network interfaces (errors)                        |                                     |
|                                       | NFS                                                          | NFS client                                         |                                     |
|                                       | NFSD                                                         | NFS server                                         |                                     |
|                                       | SOCK                                                         | Sockets                                            | (v4)                                |
|                                       | IP                                                           | IP traffic                                         | (v4)                                |
|                                       | EIP                                                          | IP traffic                                         | (v4) (errors)                       |
|                                       | ICMP                                                         | ICMP traffic                                       | (v4)                                |
|                                       | EICMP                                                        | ICMP traffic                                       | (v4) (errors)                       |
|                                       | TCP                                                          | TCP traffic                                        | (v4)                                |
|                                       | ETCP                                                         | TCP traffic                                        | (v4) (errors)                       |
|                                       | UDP                                                          | UDP traffic                                        | (v4)                                |
|                                       | SOCK6                                                        | Sockets                                            | (v6)                                |
|                                       | IP6                                                          | IP traffic                                         | (v6)                                |
|                                       | EIP6                                                         | IP traffic                                         | (v6) (errors)                       |
|                                       | ICMP6                                                        | ICMP traffic                                       | (v6)                                |
|                                       | EICMP6                                                       | ICMP traffic                                       | (v6) (errors)                       |
|                                       | UDP6                                                         | UDP traffic                                        | (v6)                                |
|                                       | FC                                                           | Fibre channel HBAs                                 |                                     |
|                                       | SOFT                                                         | Software-based network processing                  |                                     |



See more:
“ SYSSTAT Howto：Linux服务器的部署和配置指南”。Linux.com | August 10, 2009
https://www.linux.com/training-tutorials/sysstat-howto-deployment-and-configuration-guide-linux-servers/

更多相关工具
https://wiki.archlinux.org/index.php/List_of_applications/Utilities#System_monitors































































