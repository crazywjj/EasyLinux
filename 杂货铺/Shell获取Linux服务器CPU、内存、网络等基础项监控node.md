# shell获取Linux服务器CPU、内存、网络等基础项监控node



**一、获取系统（CPU）负载**shell

系统平均负载的定义：在特定时间间隔内运行队列中的平均进程数。缓存

如何获取系统（CPU）负载？bash

在Linux系统中能够经过/proc/loadavg文件获取，其中各项参数含义解释：服务器

```
   0.11 0.09 0.06 2/263 10982
```



 前三个数字分别表示：一、五、15分钟的系统负载（或者叫平均进程数）， 第四个相似分数形势的分别表示正在运行的进程数和系统当前总的进程数，最后一个数字表示的最近一个运行进程的ID。网络

 若是咱们想要获取咱们当前系统的CPU负载使用状况能够经过下边的shell命令获取：ide

```
   cat /proc/loadavg | awk ‘{print $1,$2,$3}’
```

除了经过查看/proc/loadavg获取系统的负载，还可使用Linux下的uptime、w、top等命令来获取，当使用这些命令时，输出结果中都会有load average字样，那么后边跟的三个数字就是当前系统在一、五、15分钟内的平均负载。性能

通常来讲当每一个CPU的当前进程数（运行队列长度）持续大于1，就须要开始调查引发问题的缘由了，若是每一个CPU的任务数持续大于3，就须要完全检查解决这个问题了，由于这个时候任何一个进程运行时都不能立马获得CPU的响应。若是每一个CPU的任务数大于5，那么就说明你的服务出现了严重问题，若是不及时处理，可能会致使宕机。spa

**二、获取CPU使用率**接口

CPU使用率是指当前运行的程序（进程）占用的CPU资源状况，也就是说CPU利用率是一个程序占用一个CPU处理器多少时间的百分比。在Linux/Unix下，CPU利用率又分为用户态、系统态和空闲态，分别用来表示CPU处于用户态执行的时间、系统内核执行的时间和空闲系统进程的时间。在一般咱们所说的CPU利用是指：CPU执行非系统空闲进程的时间/CPU总的执行时间。

  不要与CPU负载混淆，CPU利用率和CPU负载在必定程度上都能用在衡量一个服务器的资源使用状况，但CPU负载和CPU使用率之间没有绝对的关联关系，更不能混为一谈，关于CPU负载（系统负载）的概念在上节中已经说明。

如何获取CPU使用率？

在Linux系统中能够经过/proc/stat文件来计算CPU的使用率。其中各项参数说明：



```
cpu  13020415 2752 14585234 1898039188 5345 0 46019 0 0 0
cpu0 3212795 820 3641656 474590184 756 0 9239 0 0 0
cpu1 3285261 628 3634202 474396971 3220 0 8799 0 0 0
cpu2 3388112 637 3695487 474389928 420 0 3115 0 0 0
cpu3 3134246 665 3613888 474662103 947 0 24864 0 0 0
intr 4669596224 155 10 0 0 0 0 2 0 1713 0 0 0 16 0 0 4715099 0 535144 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 8955035 7121118 7194150 7163302 0 1739264 7 2 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
ctxt 4454886169
btime 1524214872
processes 3187040
procs_running 1
procs_blocked 0
softirq 1113211729 0 591606805 1931306 31654758 2851576 0 4047 296149644 0 189013593
```



以个人机器为例：

就拿第一行来讲：

```
   cpu  13020415 2752 14585234 1898039188 5345 0 46019 0 0 0
```



第一个字段(cpu)是cpu标识。

第二个字段（13020415）表示是从系统启动开始一直到当前时刻，进程在用户态(user)下执行的时间积累。

第三个字段(2752)表示是从系统启动开始一直到当前时刻，nice值为负的进程所占用的CPU时间。

第四个字段(14585234)表示是从系统启动开始一直到当前时刻，进程在系统内核(system)的执行时间积累。

第五个字段(1898039188)表示是从系统启动开始一直到当前时刻，处硬盘IO等待之外其余的空闲时间（idle）积累。

第六个字段（5345）表示是从系统启动开始一直到当前时刻，IO等待(iowait)的时间积累。

第七个字段(0)表示是从系统启动开始一直到当前时刻，硬中断的时间(irq)。

第八个字段(46019)表示是从系统启动开始一直到当前时刻，软中断的时间（softirq）。

剩下的几行：

intr: 给出的是中断信息，第一个数为自系统启动以来，发生的全部中断的次数；后面的每一个数对应一个特定的中断，表示该中断自系统启动以来发生的次数。

ctxt: 表示系统CPU发生的上下文交换次数。

btime: 表示自系统启动到如今的时间，单位为秒。

processes: 表示自系统启动以来所建立的任务的数目。

procs_running: 当前运行队列的任务数目。

procs_blocked: 当前被阻塞的任务数目，等待I/O完成次数。

CPU的使用率能够经过以下方式计算：

cpu_usage=(idle2-idle1)/(cpu2-cpu1)*100

cpu_usage=[(user2+sys2+nice2)-(user1+sys1+nice1)]/(total2-total)*100

**获取CPU使用率的相关脚本以下：**



```
#!/bin/sh
#
#脚本功能描述：依据/proc/stat文件获取并计算CPU使用率
#
#CPU时间计算公式：CPU_TIME=user+system+nice+idle+iowait+irq+softirq
#CPU使用率计算公式：cpu_usage=(idle2-idle1)/(cpu2-cpu1)*100
#默认时间间隔
TIME_INTERVAL=5
 
LAST_CPU_INFO=$(cat /proc/stat | grep -w cpu | awk '{print $2,$3,$4,$5,$6,$7,$8}')
LAST_SYS_IDLE=$(echo $LAST_CPU_INFO | awk '{print $4}')
LAST_TOTAL_CPU_T=$(echo $LAST_CPU_INFO | awk '{print $1+$2+$3+$4+$5+$6+$7}')
sleep ${TIME_INTERVAL}
NEXT_CPU_INFO=$(cat /proc/stat | grep -w cpu | awk '{print $2,$3,$4,$5,$6,$7,$8}')
NEXT_SYS_IDLE=$(echo $NEXT_CPU_INFO | awk '{print $4}')
NEXT_TOTAL_CPU_T=$(echo $NEXT_CPU_INFO | awk '{print $1+$2+$3+$4+$5+$6+$7}')
 
#系统空闲时间
SYSTEM_IDLE=`echo ${NEXT_SYS_IDLE} ${LAST_SYS_IDLE} | awk '{print $1-$2}'`
#CPU总时间
TOTAL_TIME=`echo ${NEXT_TOTAL_CPU_T} ${LAST_TOTAL_CPU_T} | awk '{print $1-$2}'`
CPU_USAGE=`echo ${SYSTEM_IDLE} ${TOTAL_TIME} | awk '{printf "%.2f", 100-$1/$2*100}'`
 
echo "CPU Usage:${CPU_USAGE}%"
```



除了利用/proc/stat文件获取CPU使用率，还能够经过top等命令获取CPU的使用率，如

```
# top -n 1 | grep Cpu
 %Cpu(s):  1.5 us,  1.5 sy,  0.0 ni, 96.9 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
```



**三、获取内存使用状况**

1）在Linux下要查看内存的使用状况通常使用free命令：



```
# free
total       used       free     shared    buffers     cached
Mem:       3922048    3416120     505928       1396     137396    2836636
-/+ buffers/cache:     442088    3479960
Swap:      8241148       3500    8237648
```



**其中的各项字段解释说明：**

total: 物理内存的总大小

used: 已经使用的内存大小

free：空闲的内存大小

shared：应用程序共享的内存

buffers: 缓存，主要用于目录方面，inode值等

cached: 缓存，主要用于已打开的文件

经过上边的解释说明咱们能够获得：

total = used + free

used = shared + buffers + cached

因此在计算系统内存使用率是时候，从系统的层面考虑，能够直接经过mem usage = used / total 计算内存使用率。

 

而第三行(-/+buffers/cache)中主要是针对应用程序的内存使用来讲的：

-buffers/cache:应用程序使用的内存大小，即used减去缓存值

+buffers/cache:全部可供应用程序使用的内存大小，free加上缓存值

从应用程序角度看，对于应用程序来讲buffers/cache也等同于可用的，由于buffers/cache自己是为了提升文件的读取性能，因此当内存紧张的时候，buffers/cache会被回收，从新分配给申请内存的应用程序。

 所以从应用程序的角度来讲，可用内存 = 系统free + buffers + cached

 

2）除了经过free命令获取Linux内存的使用状况，咱们还能够经过/proc/meminfo文件来获取内存的使用率：



```
#cat /proc/meminfo
MemTotal:        1883388 kB
MemFree:           81952 kB
MemAvailable:     995960 kB
Buffers:            2072 kB
Cached:           747400 kB
SwapCached:            4 kB
Active:           520372 kB
Inactive:         599848 kB
Active(anon):     173680 kB
Inactive(anon):   296344 kB
Active(file):     346692 kB
Inactive(file):   303504 kB
...
```

**相关字段解释说明：**

MemTotal：全部物理可用RAM内存

MemFree：当前空闲内存

MemAvailable: 预计可以使用的内存

Buffers: 用来给文件作缓冲的内存

Cached: 高速缓冲存储器所用内存

SwapCached: 交换空间

Active: 活跃使用的Buffer或Cache的内存大小

Inactive:不常用的Buffer或Cache的内存大小



**###利用/proc/meminfo获取内存使用状况的脚本以下：**



```
#!/bin/sh
mem_use_info=(`awk '/MemTotal/{memtotal=$2}/MemAvailable/{memavailable=$2}END{printf "%.2f %.2f %.2f",memtotal/1024/1024," "(memtotal-memavailable)/1024/1024," "(memtotal-memavailable)/memtotal*100}' /proc/meminfo`)
 
echo total:${mem_use_info[0]}G  used:${mem_use_info[1]}G  Usage:${mem_use_info[2]}%
```



 

**四、获取实时网卡流量信息**

  在Linux下若是须要查看实时流量通常都采用iftop命令来查看不一样网卡的实时流量信息。

  可是若是想分析网卡的网络包、流量、错包、丢包等信息通常都是经过/proc/net/dev文件来分析。

 针对/proc/net/dev文件，一般文件内容以下：



```
#cat /proc/net/dev
Inter-|   Receive                                                |  Transmit
 face |bytes    packets errs drop fifo frame compressed multicast|bytes    packets errs drop fifo colls carrier compressed
ens192: 110400390 1840002    0  904    0     0          0         0      900      14    0    0    0     0       0          0
    lo:  923561   14578    0    0    0     0          0         0   923561   14578    0    0    0     0       0          0
ens160: 1375841938 17880629    0  909    0     0          0         0 1331915939 15766579    0    0    0     0       0          0
```

**相关字段解释说明:**

从第一行中能够看出这个文件大体分为三部分，分别是：

第一个字段表示接口名称

第二个字段Recevice表示收包

第三个字段Transmit表示发包。

下边几行内容相同：

第一个字段(face)：表示接口名称

第二个字段和第十个字段(byte)：分别表示收到、发送的字节数

第三个字段和第十一个字段(packets)：分别表示收到、发送的正确数据包的数量

第四个字段和第十二个字段(errs)：分别表示收到、发送的错误数据包量

第五个字段和第十三个字段（drop）：分别表示收到、发送丢弃的数据包量

 

**下边是获取服务器网卡实时流量的脚本：**



```
#!/bin/sh

#经过/proc/net/dev获取执行网卡的信息

if [ "$1" = "" ];then  #判断后面是否有跟参数
  echo -e "\n   use interface_name after the script,like \"script eth0\"...\n"
  exit -1
fi

echo -e "\n   start monitoring the $1,press \"ctrl+c\" to stop"
echo ----------------------------------------------------------

#时间间隔（频率）
interval=10

file=/proc/net/dev  #内核网卡信息文件
while true
  do
  rx_bytes=`cat $file|grep $1|sed 's/^ *//g'|awk -F '[ :]+' '{print $2}'`  #这里sed>这一步为了同时兼容centos6和7
  tx_bytes=`cat $file|grep $1|sed 's/^ *//g'|awk -F '[ :]+' '{print $10}'`
  sleep 10
  rx_bytes_later=`cat $file|grep $1|sed 's/^ *//g'|awk -F '[ :]+' '{print $2}'`
  tx_bytes_later=`cat $file|grep $1|sed 's/^ *//g'|awk -F '[ :]+' '{print $10}'`

  # bytes/1024=kb
  speed_rx=`echo "scale=2;($rx_bytes_later - $rx_bytes)/1024/$interval"|bc`
  speed_tx=`echo "scale=2;($tx_bytes_later - $tx_bytes)/1024/$interval"|bc`

  printf "%-3s %-3.1f %-10s %-4s %-3.1f %-4s\n" in: $speed_rx kb/s out: $speed_tx kb/s
done

```











