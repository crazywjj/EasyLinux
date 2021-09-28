[TOC]







# 第四单元 FTP服务



# 4.1 FTP 服务概述

**FTP** 是`File Transfer Protocol`（**文件传输协议**）的英文简称，它**工作在OSI模型的第七层**，TCP模型的第四层上，即传输，使用TCP传输而不是UDP，这样FTP客户在和服务器建立连接前就要经过一个被广为熟知的”三次握手”的过程，它带来的意义在于客户与服务器之间的连接是可靠的，而且是面向连接，为数据的传输提供了可靠的保证。

FTP服务使用FTP协议（文件传输协议）来进行**文件的上传和下载**，可以非常方便的进行远距离的文件传输，还支持断点续传功能，可以大幅度地减小CPU和网络带宽的开销，并实现相应的安全控制。



ps:主动模式要求客户端和服务器端同时打开并且监听一个端口以建立连接。在这种情况下，客户端由于安装了防火墙会产生一些问题。所以，创立了被动模式。被动模式只要求服务器端产生一个监听相应端口的进程，这样就可以绕过客户端安装了防火墙的问题。

FTP和NFS、Samba ：三大文件共享服务器。

FTP软件：**vsftpd**,wu-ftp,proftp等

最常用的FTP服务器架设使用vsftpd软件 ：安全“ very secure”



## 4.1.1 FTP两种工作模式及原理

ftp协议的连接方式有两种，一种是命令连接，一种是数据连接，而ftp的数据连接模式也有两种，一种是主动模式，一种是被动模式。

FTP会话连接时包含了两个通道，一个叫**控制通道，端口号21**；一个叫**数据通道，端口号20**。 

控制通道：控制通道是和FTP服务器进行沟通的通道，连接FTP，**发送FTP指令都是通过控制通道来完成的**。 

数据通道：数据通道是和FTP服务器进行**文件传输或者列表的通道**。



FTP协议中，**控制连接均有客户端发起**，而数据连接有两种工作方式：PORT方式和PASV方式

1.FTP的PORT（主动模式）和PASV（被动模式）

(1) PORT（主动模式）

PORT中文称为主动模式，工作的原理： 

FTP客户端连接到FTP服务器的21端口→发送用户名和密码登录，登录成功后要list列表或者读取数据时→客户端随机开放一个端口（1024以上）→发送 PORT命令到FTP服务器，告诉服务器客户端采用**主动模式**并开放端口→FTP服务器收到PORT主动模式命令和端口号后，通过服务器的20端口和客户端开放的端口连接，发送数据，原理如下图：

![1567049809488](assets/1567049809488.png)



(2) PASV（被动模式）

PASV是Passive的缩写，中文成为被动模式，工作原理：

FTP客户端连接到FTP服务器的21端口→发送用户名和密码登录，登录成功后要list列表或者读取数据时→发送PASV命令到FTP服务器→ 服务器在本地随机开放一个端口（1024以上）→然后把开放的端口告诉客户端， 客户端再连接到服务器开放的端口进行数据传输，原理如下图：

![1567049833778](assets/1567049833778.png)



## 4.1.2 vsftpd软件

### 4.1.2.1 vsftpd简介

软件：vsftpd

服务名：vsftpd

配置文件：/etc/vsftpd/vsftpd.conf



### 4.1.2.2 安装vsftpd服务端和客户端

```shell
yum install -y vsftpd   #服务端
yum install -y lftp		#客户端
/etc/init.d/vsftpd start
chkconfig vsftpd on

# 关闭防火墙和Selinux
service iptables stop
setenforce 0
sed -i '/^SELINUX=/s#SELINUX=.*#SELINUX=disabled#g' /etc/sysconfig/selinux
```



### 4.1.2.3 配置文件结构

```shell
vsftpd的核心文件和目录：
/etc/pam.d/vsftpd 			#基于PAM认证的vsftpd验证配置文件
/etc/logrotate.d/vsftpd 	#日志轮转备份配置文件
/etc/rc.d/init.d/vsftpd 	#vsftpd启动脚本，供server调用
/etc/vsftpd 				#vsftpd的主目录
/etc/vsftpd/ftpusers 		#默认的黑名单
/etc/vsftpd/user_list 		#指定允许使用vsftpd的用户列表文件
/etc/vsftpd/vsftpd.conf 	#vsftpd主配置文件
/var/ftp 					#vsftpd默认共享目录(匿名用户的根目录)
/etc/vsftpd/vsftpd_conf_migrate.sh 	#是vsftpd操作的一些变量和设置脚本
```

vsftp默认传输方式默认是PORT模式（主动模式）

```
port_enable=YES|NO
```

被动模式常用配置：

```bash
anonymous_enable=NO
listen_port=61022
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
connect_from_port_20=YES
accept_timeout=60
connect_timeout=60
idle_session_timeout=300
xferlog_enable=YES
xferlog_std_format=YES
pam_service_name=vsftpd
userlist_enable=YES
xferlog_file=/var/log/xferlog
vsftpd_log_file=/var/log/vsftpd.log
tcp_wrappers=YES
listen=YES
listen_ipv6=NO
chroot_local_user=YES
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd/chroot_list
allow_writeable_chroot=YES
seccomp_sandbox=NO
pasv_enable=YES
pasv_min_port=61030
pasv_max_port=61033
max_per_ip=2
#pasv_address=59.255.61.34
```





### 4.1.2.4 vsftp的3种远程的登录方式

（1）匿名用户登录方式 
　　就是不需要用户名和密码。就能登录到服务器vsftp（默认只有下载权限）
（2）本地用户方式 
　　需要帐户名和密码才能登录。而且，这个帐户名和密码，都是在你linux系统里面，已经有的用户。 
（3）虚拟用户方式 
　　同样需要用户名和密码才能登录。但是和上面的区别就是，这个用户名和密码，在你linux系统中是没有的(没有该用户帐号)

### 4.1.2.5 vsftpd默认配置

```shell
[root@ c6m01 vsftpd]# cp vsftpd.conf{,.bak}
[root@ c6m01 vsftpd]# egrep -v '^$|^#' vsftpd.conf
anonymous_enable=YES
local_enable=YES
write_enable=YES
cal_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=YES
pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
```

### 4.1.2.6 vsftpd配置文件参数详解

**匿名用户相关参数**

```bash
anonymous_enable=YES    # 是否允许匿名用户登陆
no_anon_password=NO     # 是否忽略对匿名用户的密码检测
anon_root               # 匿名登陆后尝试更改的目录
ftp_username=ftp        # 匿名用户登陆的username
anon_mkdir_write_enable=NO    # 是否允许匿名用户创建目录
anon_other_write_enable=NO    # 是否允许匿名用户对文件增、删、改
anon_upload_enable=NO         # 是否允许匿名用户上传文件
anon_world_readable_only=YES  # 如果为YES，匿名用户只能下载可读的文件
anon_max_rate=0               # 允许匿名用户最大数据传输速率，单位为B/s
anon_umask=077
chown_uploads=NO              # 如果设置为YES，所有的匿名用户上传的文件属主都设置为chown_username指定用户
chown_username=root           # 指定chown_uploads中所需的文件属主用户
deny_email_enable=NO          # 如果设置为YES，则提供一个内容为mail address的文件，若是使用匿名登入，则会要求输入email address，如果它不在文件内，则禁止进入
banned_email_file=/etc/vsftpd/banned_emails # 指定deny_email_enable中的文件路径
```

**本地用户相关参数**

```bash
local_enable=YES     # 是否允许本地用户登陆
write_enable=YES     # 本地用户写权限
local_umask=022      # 本地用户的文件umask码
local_root=none      # 本地用户登陆后改变的目录，默认是各自家目录
local_max_rate=0(unlimited)     # 本地用户使用的最大传输速度，单位为B/s
```

**虚拟用户相关参数**

```bash
pam_service_name=vsftpd      # 设置PAM使用的名称
guest_enable=NO              # 是否启用虚拟用户
guest_username=vsftpuser     # 指定本地用户名，用来映射虚拟用户
virtual_use_local_privs=NO   # 如果启用，虚拟用户将使用与本地用户相同的权限，否则与匿名用户权限相同
```

**banner参数**

```bash
dirmessage_enable=YES      # 如果设置为YES，当用户首次进入一个目录后，会查找.message文件并显示。
message_file=.message      # 设置message文件
ftpd_banner=Welcome to blah FTP service.  #设置进入banner信息
banner_file=none        # 设置一个包含banner信息的文件路径，开启它将会覆盖ftpd_banner
```

**数据传输模式**

```bash
ascii_upload_enable=YES    # 是否启用ASCII模式上传数据
ascii_download_enable=YES  # 是否启用ASCII模式下载数据
```

**工作模式与端口设置**

```bash
listen_port=21              # 设置ftp服务工作的端口
connect_from_port_20=YES    # 主动模式数据传输的端口
pasv_enable=YES             # YES为被动模式工作，NO则是主动模式，默认YES
pasv_max_port=0             # PASV模式下，最大传输数据端口号，0为任意无限制
pasv_min_port=0             # PASV模式下，最小传输数据端口号
```

**超时时间设置**

```bash
idle_session_timeout=600       # 客户端连接FTP后无命令超时时间
data_connection_timeout=120    # 无数据传输超时时间
accept_timeout=60              # 建立FTP连接的超时时间
connect_timeout=60             # PORT模式下建立数据连接的超时时间
```

**目录权限，锁定家目录**

```bash
chroot_local_user=NO     # 是否禁止本地用户切换到家目录上级目录，绑定家目录为用户的根目录 限制所有用户都在家目录
chroot_list_enable=NO    # 是否启用chroot列表文件，写入文件中的用户将锁定家目录
chroot_list_file=/etc/vsftpd/chroot_list       # 指定用户列表文件的文件路径

chroot_local_user=YES，chroot_list_enable=YES  # chroot_list 文件中的用户可以切换到其他目录
chroot_local_user=NO，chroot_list_enable=YES   # chroot_list_file文件中的用户将锁定在家目录下
chroot_local_user=YES，chroot_list_enable=NO   # 所有本地用户都锁定在家目录下
chroot_local_user=NO，chroot_list_enable=NO    # 所有本地用户都可以切换到其他目录
```

**主服务运行模式与连接设置**

```bash
listen=YES
# 设 置vsftpd服务器是否以standalone模式运行，有很多与服务器运行相关的配置命令，需要在此模式下才有效
# 若设置为NO，则vsftpd以super daemon运行，要受到xinetd 服务的管控，功能上会受到限制max_clients=0(unlimited)
# 设置客户端最大连接数，standalone模式下有效max_per_ip=0(unlimited)
# 同一IP客户端最大连接数，standalone模式下有效
```

**用户访问控制**

```bash
userlist_file=/etc/vsftpd/user_list      # 控制用户访问FTP的文件
userlist_enable=NO      # 如果启用，vsftpd将从userlist_file提供的文件名加载用户名列表
userlist_deny=YES       # 如果设为YES，userlist_file文件中的用户不可以访问FTP服务；如果设为NO，只用文件中用户才能访问服务
/etc/vsftpd/ftpusers    # 文件用来定义不允许访问FTP服务的用户列表，优先级比userlist_deny高
```

**自定义用户配置**

```bash
user_config_dir=/etc/vsftpd/virtual_user_conf/  # 指定虚拟用户配置文件目录，目录下可以创建与虚拟用户名相同的文件，给予不同的权限设置
```

**其他设置**

```bash
download_enable=YES   # 如果设置为NO，所有下载请求被拒绝
ls_recurse_enable=NO  # 启用后，允许使用ls -R命令，此命令可能会消耗大量资源
```



**测试访问：**

**验证结果：匿名用户权限：默认只可以下载**

Windows资源管理器输入：ftp://10.0.0.21/

![1567064195437](assets/1567064195437.png)

测试上传的权限：

![1567064263921](assets/1567064263921.png)

测试删除权限：

![1567064311632](assets/1567064311632.png)





# 4.2 匿名用户登录

## 4.2.1 匿名用户常用参数

```shell
anon_root=/var/ftp				#匿名用户默认共享目录
anon_upload_enable=YES 			#启用匿名用户上传文件的功能（文件夹不成）
anon_mkdir_write_enable=YES 	#开放匿名用户写和创建目录的权限（管理员权限）
anon_other_write_enable=YES 	#允许匿名用户重命名，删除

[root@ c6m01 vsftpd]# cat vsftpd.conf
#匿名用户权限相关
anonymous_enable=YES
anon_root=/var/ftp
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
####
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=YES
pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES

[root@ c6m01 ftp]# /etc/init.d/vsftpd restart
```

## 4.2.2 测试匿名用户登录

所在文件夹要有写入权限 例如：`chmod -R o+w /var/ftp/ `

匿名用户家目录/var/ftp权限是755，这个权限是不能改变的。

![1567065435709](assets/1567065435709.png)





# 4.3 本地用户登录

## 4.3.1 创建本地用户tom并设置密码

如果tom用户已存在则不需要创建

```shell
useradd   tom
echo '123456'|passwd --stdin tom
```

## 4.3.2 本地用户常用参数

```shell
local_enable=YES	#设定本地用户可以访问
write_enable=YES	#全局设置，是否容许写入（无论是匿名用户还是本地用户，若要启用上传权限的话，就要开启他）
local_umask=022 	#设定上传后文件的权限掩码
local_root=/home/tom		#本地用户ftp根目录，默认是本地用户的家目录
local_max_rate=0			#本地用户最大传输速率（字节）。0为不限制
```

## 4.3.3 全局配置文件

```bash
cat /etc/vsftpd/vsftpd.conf


anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES
pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
chroot_local_user=YES
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd/chroot_list
allow_writeable_chroot=YES


echo " " >>/etc/vsftpd/chroot_list
#确保/etc/vsftpd/chroot_list为空，否则会有安全隐患。
```

安全修复：

1、vsftpd服务用户可使用其他方式登录操作系统

风险描述
默认情况下专用于ftp访问的用户没有登录操作系统的权限。如果专用于ftp访问的用户可使用其他方式登录操作系统，会导致攻击者利用ftp用户执行许多系统命令或使用系统的漏洞进行权限提升。当vsftp用户可使用其他方式登录系统会为攻击者提供极大便利，而且不易被管理员发现。

如果vsftp使用操作系统帐号进行身份认证，则检查用于ftp认证的操作系统帐号中是否存在能够使用其他方式登录（如ssh）系统的帐号
vsftpd账号允许登录操作系统的用户为：tom；

修复建议
设置用户ftp登录的操作系统帐号的bash为/sbin/nologin，或锁定该帐号。设置账号不可登录系统后，发现ftp无法登录，修改

```bash
 vim /etc/pam.d/vsftpd

注释如下配置
#auth       required	pam_shells.so
```





## 4.3.4 测试本地用户上传下载

**使用winscp客户端**

![1567330500061](assets/1567330500061.png)



![1567330557462](assets/1567330557462.png)



# 4.4 FTP客户端

## 4.4.1 Linux下FTP客户端

```shell
1. yum install -y lftp

在linux端使用ftp或者lftp命令登录vsftpd服务
lftp使用介绍
lftp 是一个功能强大的下载工具，它支持访问文件的协议: ftp, ftps, http, https, hftp, fish.(其中ftps和https需要在编译的时候包含openssl库)。lftp的界面非常想一个shell: 有命令补全，历史记录，允许多个后台任务执行等功能，使用起来非常方便。它还有书签、排队、镜像、断点续传、多进程下载等功能。

2. lftp登录：

#登录到ftp--法1
lftp (ftp://)user:password@site:21  #ftp://可以省略，默认21端口可以省略
#登录到ftp--法2
lftp (ftp://)user@site:port   #这种方式回车后，系统提示输入密码
#登录到sftp---法1
lftp sftp://user:password@site:22  #如果是默认端口22，可以省略，如果不是就必须填写端口号
#登录到sftp---法2
lftp sftp://user@password:port

3. lftp常用命令：
ls  	显示远端文件列表(!ls 显示本地文件列表)。 
cd  	切换远端目录(lcd 切换本地目录)。 
get 	下载远端文件。 
mget 	下载远端文件(可以用通配符也就是 *)。 
pget 	使用多个线程来下载远端文件, 预设为五个。 
mirror 	下载/上传(mirror -R)/同步 整个目录。 
put 	上传文件。 
mput 	上传多个文件(支持通配符)。 
mv 		移动远端文件(远端文件改名)。 
rm 		删除远端文件。 
mrm 	删除多个远端文件(支持通配符)。 
mkdir 	建立远端目录。 
rmdir 	删除远端目录。 
pwd 	显示目前远端所在目录(lpwd 显示本地目录)。 
du 		计算远端目录的大小 
! 		执行本地 shell的命令(由于lftp 没有 lls, 故可用 !ls 来替代) 
lcd 	切换本地目录 
lpwd 	显示本地目录 
alias 	定义别名 
bookmark 	设定书签。 
exit 		退出ftp
```

注意点：

1. ftp只能上传和下载文件，不能对文件夹进行操作，如果想上传/下载文件夹需要进行压缩/解压缩操作

2. ftp服务器登录通常使用匿名登录方式(用户名：`anonymous`或者`ftp`，匿名用户只能在指定目录范围内登录)

3. lftp第三方ftp客户端，可以进行目录操作

![1567869702118](assets/1567869702118.png)



## 4.4.2 Windows下FTP客户端

1.个人喜欢xftp和winscp工具。

2.windows自带的资源管理器也可以登录ftp

![1567330816772](assets/1567330816772.png)



# 4.5 FTP限速等参数	

```shell
max_clients=50 	#设置vsftpd允许的最大连接数，默认值为0，表示不受限制。若设置为100时，则同时允许有100个连接，超出的将被拒绝。只有在standalone模式运行才有效。 
max_per_ip=10 	#设置每个IP允许与FTP服务器同时建立连接的数目。默认值为0，表示不受限制。只有在standalone模式运行才有效。 
anon_max_rate=10M 	#设置匿名登入者使用的最大传输速度，单位为B/s，0表示不限制速度。默认值为0。 
local_max_rate=10M 	#本地用户使用的最大传输速度，单位为B/s，0表示不限制速度。预设值为0。 
```



