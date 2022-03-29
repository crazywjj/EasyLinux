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

```bsh
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