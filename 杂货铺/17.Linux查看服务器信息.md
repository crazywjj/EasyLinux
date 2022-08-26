







# Linux查看服务器信息



在 `Linux` 系统获取物理机器的信息：

我们可以通过 `lshw` 或者 `dmidecode` 来获取服务器对应的物理机信息。

我们这里还是推荐使用 `dmidecode` 来获取。

安装 `dmidecode`

```
yum install -y dmidecode
```

# 一、查看系统信息(包含机器型号)

# 1.1 查看机型和品牌

```bash
[root@djx ~]# dmidecode  --type system
# dmidecode 3.1
Getting SMBIOS data from sysfs.
SMBIOS 3.2.0 present.
# SMBIOS implementations newer than version 3.1.1 are not
# fully supported by this version of dmidecode.

Handle 0x0100, DMI type 1, 27 bytes
System Information
	Manufacturer: Dell Inc.         # 机器品牌
	Product Name: PowerEdge R740    # 机器型号
	Version: Not Specified
	Serial Number: FW89BX2
	UUID: 4c4c4534-0057-3810-8019-c6c04f425832
	Wake-up Type: Power Switch
	SKU Number: SKU=NotProvided;ModelName=PowerEdge R740
	Family: PowerEdge
#直接获取机型和品牌、序列号 dmidecode  --type system |grep -E 'Manufacturer|Product Name'
#lshw|head  #可以获取物理机机器型号和
```

# 二、查看CPU 信息

## 2.1 查看CPU 型号

```bash
[root@djx ~]# dmidecode --type processor
# dmidecode 3.1
Getting SMBIOS data from sysfs.
SMBIOS 3.2.0 present.
# SMBIOS implementations newer than version 3.1.1 are not
# fully supported by this version of dmidecode.

Handle 0x0400, DMI type 4, 48 bytes
Processor Information
	Socket Designation: CPU1
	Type: Central Processor
	Family: Xeon
	Manufacturer: Intel
	ID: 57 06 04 00 FF F5 EB BF
	Signature: Type 0, Family 6, Model 85, Stepping 7
	Flags:
		FPU (Floating-point unit on-chip)
		VME (Virtual mode extension)
		DE (Debugging extension)
		PSE (Page size extension)
		TSC (Time stamp counter)
		MSR (Model specific registers)
		PAE (Physical address extension)
		MCE (Machine check exception)
		CX8 (CMPXCHG8 instruction supported)
		APIC (On-chip APIC hardware supported)
		SEP (Fast system call)
		MTRR (Memory type range registers)
		PGE (Page global enable)
		MCA (Machine check architecture)
		CMOV (Conditional move instruction supported)
		PAT (Page attribute table)
		PSE-36 (36-bit page size extension)
		CLFSH (CLFLUSH instruction supported)
		DS (Debug store)
		ACPI (ACPI supported)
		MMX (MMX technology supported)
		FXSR (FXSAVE and FXSTOR instructions supported)
		SSE (Streaming SIMD extensions)
		SSE2 (Streaming SIMD extensions 2)
		SS (Self-snoop)
		HTT (Multi-threading)
		TM (Thermal monitor supported)
		PBE (Pending break enabled)
	Version: Intel(R) Xeon(R) Silver 4208 CPU @ 2.10GHz  # CPU 型号
	Voltage: 1.8 V
	External Clock: 9600 MHz
	Max Speed: 4000 MHz
	Current Speed: 2100 MHz
	Status: Populated, Enabled
	Upgrade: Socket LGA2011
	L1 Cache Handle: 0x0700
	L2 Cache Handle: 0x0701
	L3 Cache Handle: 0x0702
	Serial Number: Not Specified
	Asset Tag: Not Specified
	Part Number: Not Specified
	Core Count: 8
	Core Enabled: 8
	Thread Count: 16
	Characteristics:
		64-bit capable
		Multi-Core
		Hardware Thread
		Execute Protection
		Enhanced Virtualization
		Power/Performance Control

Handle 0x0401, DMI type 4, 48 bytes
Processor Information
	Socket Designation: CPU2
	Type: Central Processor
	Family: Unknown
	Manufacturer: Not Specified
	ID: 00 00 00 00 00 00 00 00
	Version: Not Specified
	Voltage: Unknown
	External Clock: Unknown
	Max Speed: 4000 MHz
	Current Speed: Unknown
	Status: Unpopulated
	Upgrade: Socket LGA2011
	L1 Cache Handle: Not Provided
	L2 Cache Handle: Not Provided
	L3 Cache Handle: Not Provided
	Serial Number: Not Specified
	Asset Tag: Not Specified
	Part Number: Not Specified
	Characteristics: None
#  dmidecode --type processor  |grep 'Version'
```

## 2.2 查看CPU的物理数量

```bash
cat  /proc/cpuinfo |grep "physical id"|sort|uniq|wc -l
```

## 2.3 查看 CPU核心数量（非逻辑CPU）

```bash
grep "cpu cores" /proc/cpuinfo | uniq | awk -F ":" "{print $2}"
```

## 2.4 查看 CPU数量(逻辑)

```bash
cat  /proc/cpuinfo |grep 'processor'|wc -l
```

## 2.5 查看CPU的支持的最大内存

```bash
我这个没有找到，可以从 cpu 型号进行查询。 直接百度搜索该CPU信息，然后查看该 cpu 的参数。
```

## 2.6 lscpu详解

## lscpu 命令详解

| 关键词             | 详解               |
| ------------------ | ------------------ |
| Architecture       | #架构              |
| CPU(s)             | #逻辑cpu个数       |
| Thread(s) per core | #每个核心线程数    |
| Core(s) per socket | #每个物理CPU的核数 |
| Socket(s)          | #物理CPU个数       |
| CPU MHz            | #cpu主频           |





```bash
[root@node1 ~]# lscpu
Architecture:          x86_64   #架构
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                32    #逻辑cpu个数
On-line CPU(s) list:   0-31
Thread(s) per core:    2   #每个核心线程数
Core(s) per socket:    8   #每个物理CPU的核数
Socket(s):             2   #物理CPU个数
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 62
Model name:            Intel(R) Xeon(R) CPU E5-2650 v2 @ 2.60GHz
Stepping:              4
CPU MHz:               1279.528
CPU max MHz:           2600.0000
CPU min MHz:           1200.0000
BogoMIPS:              5187.61
Virtualization:        VT-x
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              20480K
NUMA node0 CPU(s):     0-31
Flags:                 fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 monitor ds_cpl vmx smx est tm2 ssse3 cx16 xtpr pdcm pcid dca sse4_1 sse4_2 x2apic popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm epb ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase smep erms xsaveopt dtherm arat pln pts md_clear spec_ctrl intel_stibp flush_l1d

```





# 三、查询主板信息

## 3.1 查询主板型号

```bash
[root@djx ~]#  dmidecode --type baseboard|head
# dmidecode 3.1
Getting SMBIOS data from sysfs.
SMBIOS 3.2.0 present.
# SMBIOS implementations newer than version 3.1.1 are not
# fully supported by this version of dmidecode.

Handle 0x0200, DMI type 2, 8 bytes
Base Board Information
	Manufacturer: Dell Inc.   # 主板品牌
	Product Name: 014X06   # 生产号
```

## 3.2 查看主板可以支持的最大内存

```bash
dmidecode |grep "Maximum Capacity"
或者
dmidecode --type memory |head
```

# 四、查询磁盘相关的东西

## 4.1 磁盘大小

```
 fdisk  -l
```

## 4.2 磁盘块数

```
 fdisk  -l
 # 这个也不精确，建议直接去物理机上面看，我们下面提供戴尔服务器如何查询的方法。
```

## 4.3 是否启用raid

```
lspci   |grep  -i raid
# 这个不精确 建议安装特定的软件(不同机器不一样)进行查看
```

## 4.4 查看当前的 raid版本

```
要根据官方的文档去安装驱动进行查询。
```

# 五、查询内存条信息

## 5.1 当前内存单条大小

```
dmidecode --type memory |grep "Size"
```

## 5.2 当前多少根内存条

```
dmidecode --type memory |grep "Size"|grep 'MB'|wc -l
```

## 5.3 当前内存条型号

```
dmidecode --type memory |grep "Part Number"
```

## 5.4 可以扩展内存条数目

```
dmidecode --type memory |grep "Number Of Devices"
# 这个数据也是仅仅供参考，因为 部分服务器，单物理CPU 是可以插8根内存条，然后2个物理CPU是可以插12根内存条的，这个需要查看服务器的官方文档。
```

## 5.5 可以扩展内存总大小

```
dmidecode |grep "Maximum Capacity"
# 这个值也不是很准，跟内存条的类型与插入的内存条有关。 建议查询官方文档
```

# 六、 扩展

## 6.1 扩展信息

戴尔如何获取服务码：机器前面显示屏的字符串(7位)/或者快速服务代码。

惠普如何获取服务码： 机器上的SN 码 。HP 查询是否过保网站： https://support.hpe.com/hpsc/wc/public/home

戴尔监控物理设备一般使用的是 `IDRAC` (远程)及 `OMSA` 、 `SAE`

**内存条类型区分**

内存条类型分为 `LRDIMM`、`RDIMM`、`UDIMM`，一般来说`LRDIMM` 的内存条容量更大些。同一个机型，我们建议使用相同类型的内存条。一般只只有颗CPU的话，只能支持一半的内存条数。如何查询内存条的安装方法以及服务器支持的最大内存。我们一般从官方文档查询， 一般位于官方文档中的用户手册，技术规格-内存规格 ,安装方法为 安装文档里面内存内容

## 6.2 戴尔服务器查询是否使用了 raid

需要安装 `PERC`，官方文档。 从戴尔支持站点下载Linux PERCCLI实用程序。选择您的系统，然后按类别**SAS RAID**或通过使用关键字**PERCCLI**筛选驱动程序和下载

```
# 查看是否使用了raid，raid几
perccli /c0/v0 show all
# 查看物理磁盘数
perccli /c0/eall/sall show
```





## 1.5 Centos系统 查看RAID状态



