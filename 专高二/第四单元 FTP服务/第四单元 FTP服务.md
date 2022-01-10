[TOC]







# 第四单元 FTP文本传输协议



# 4.1 FTP 服务概述

**FTP** 是`File Transfer Protocol`（**文件传输协议**）的英文简称，它**工作在OSI模型的第七层**，TCP模型的第四层上，即传输，使用TCP传输而不是UDP，这样FTP客户在和服务器建立连接前就要经过一个被广为熟知的”三次握手”的过程，它带来的意义在于客户与服务器之间的连接是可靠的，而且是面向连接，为数据的传输提供了可靠的保证。

FTP服务使用FTP协议（文件传输协议）来进行**文件的上传和下载**，可以非常方便的进行远距离的文件传输，还支持断点续传功能，可以大幅度地减小CPU和网络带宽的开销，并实现相应的安全控制。



## 4.1.1 FTP工作模式

FTP有两种工作模式：主动模式（标准模式或standard）和被动模式(pasv模式)。FTP将命令和数据分开传输，因此FTP工作时有两个端口，命令端口和数据端口。



FTP协议中，**控制连接均有客户端发起**，而数据连接有两种工作方式：PORT方式和PASV方式。

**(1) PORT（主动模式）**

PORT中文称为主动模式，工作的原理： 

FTP客户端连接到FTP服务器的21端口→发送用户名和密码登录，登录成功后要list列表或者读取数据时→客户端随机开放一个端口（1024以上）→发送 PORT命令到FTP服务器，告诉服务器客户端采用**主动模式**并开放端口→FTP服务器收到PORT主动模式命令和端口号后，通过服务器的20端口和客户端开放的端口连接，发送数据，原理如下图：

![1567049809488](assets/1567049809488.png)



**(2) PASV（被动模式）**

PASV是Passive的缩写，中文成为被动模式，工作原理：

FTP客户端连接到FTP服务器的21端口→发送用户名和密码登录，登录成功后要list列表或者读取数据时→发送PASV命令到FTP服务器→ 服务器在本地随机开放一个端口（1024以上）→然后把开放的端口告诉客户端， 客户端再连接到服务器开放的端口进行数据传输，原理如下图：

![1567049833778](assets/1567049833778.png)

**简而言之：**

主动模式（PORT）和被动模式（PASV）。主动模式是从服务器端向客户端发起连接；被动模式是客户端向服务器端发起连接。两者的共同点是都使用 21端口进行用户验证及管理，差别在于传送数据的方式不同，PORT模式的FTP服务器数据端口固定在20，而PASV模式则在1025-65535之间随机

**主动与被动FTP优缺点：**

主动FTP对FTP服务器的管理有利，但对客户端的管理不利。因为FTP服务器企图与客户端的高位随机端口建立连接，而这个端口很有可能被客户端的防火墙阻塞掉。被动FTP对FTP客户端的管理有利，但对服务器端的管理不利。因为客户端要与服务器端建立两个连接，其中一个连到一个高位随机端口，而这个端口很有可能被服务器端的防火墙阻塞掉。



## 4.1.2 FTP服务状态码

1XX：信息 125：数据连接打开

2XX：成功类状态 200：命令OK 230：登录成功

3XX：补充类 331：用户名OK

4XX：客户端错误 425：不能打开数据连接

5XX：服务器错误 530：不能登录

附: FTP 数字代码的意义
110 重新启动标记应答。
120 服务在多久时间内ready。
125 数据链路埠开启，准备传送。
150 文件状态正常，开启数据连接端口。
200 命令执行成功。
202 命令执行失败。
211 系统状态或是系统求助响应。
212 目录的状态。
213 文件的状态。
214 求助的讯息。
215 名称系统类型。
220 新的联机服务ready。
221 服务的控制连接埠关闭，可以注销。
225 数据连结开启，但无传输动作。
226 关闭数据连接端口，请求的文件操作成功。
227 进入passive mode。
230 使用者登入。
250 请求的文件操作完成。
257 显示目前的路径名称。
331 用户名称正确，需要密码。
332 登入时需要账号信息。
350 请求的操作需要进一部的命令。
421 无法提供服务，关闭控制连结。
425 无法开启数据链路。
426 关闭联机，终止传输。
450 请求的操作未执行。
451 命令终止：有本地的错误。
452 未执行命令：磁盘空间不足。
500 格式错误，无法识别命令。
501 参数语法错误。
502 命令执行失败。
503 命令顺序错误。
504 命令所接的参数不正确。
530 未登入。
532 储存文件需要账户登入。
550 未执行请求的操作。
551 请求的命令终止，类型未知。
552 请求的文件终止，储存位溢出。
553 未执行请求的的命令，名称不正确





## 4.1.3 FTP用户认证

FTP有三种用户认证方式：

1.匿名验证，通过ftp或者anonymous登录，密码随意，则为匿名登录，匿名用户登录后会映射为ftp用户，共享位置为ftp家目录：/var/ftp，默认只有下载权限。

2.系统用户，还可以使用系统中存在的用户进行登录,默认共享目录为用户的家目录。

3.虚拟用户，自建的FTP用户则成为虚拟用户，虚拟用户的密码可以通过db文件或者mysql数据库存放，需指定映射的用户及共享目录。



# 4.2 常见 FTP 相关软件

**FTP服务器端软件**

```
Wu-ftpd,Proftpd,Pureftpd,Filezilla Server,Serv-U,Wing FTP Server,IIS
vsftpd：Very Secure FTP Daemon，CentOS 默认FTP服务器
高速，稳定，下载速度是WU-FTP的两倍
ftp.redhat.com数据:单机最多可支持15000个并发
```

**客户端软件**

```
ftp，lftp，lftpget，wget，curl
ftp -A ftpserver port -A 主动模式 –p 被动模式
lftp –u username ftpserver
lftp username@ftpserver
lftpget ftp://ftpserver/pub/file
gftp：GUI centos5 最新版2.0.19 (11/30/2008)
filezilla，FTP Rush，CuteFtp，FlashFXP，LeapFtp
IE ftp://username:password@ftpserver
```



# 4.3 Vsftpd软件

## 4.3.1 Vsftpd简介

vsftpd在linux中由 vsftpd 包提供，自centos7后不再由xinetd管理依赖于pam模块进行身份验证，配置文件：/etc/pam.d/vsftpd通过`rpm -ql vsftpd`可以查到软件包重要的文件：

软件：vsftpd

服务名：vsftpd

配置文件：/etc/vsftpd/vsftpd.conf



## 4.3.2 安装Vsftpd

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



## 4.3.3 配置文件介绍

### 4.3.3.1 配置文件结构

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

### 4.3.3.2 vsftpd默认配置

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

### 4.1.2.6 vsftpd配置文件详解

**主服务运行模式与连接设置**

```bash
listen=YES
# 设 置vsftpd服务器是否以standalone模式运行，有很多与服务器运行相关的配置命令，需要在此模式下才有效
# 若设置为NO，则vsftpd以super daemon运行，要受到xinetd 服务的管控，功能上会受到限制max_clients=0(unlimited)
# 设置客户端最大连接数，standalone模式下有效max_per_ip=0(unlimited)
# 同一IP客户端最大连接数，standalone模式下有效
```

**命令端口**

```bash
listen_port=21 #ftp命令通道端口。Default: 21
```

**主动模式配置**

```bash
connect_from_port_20=YES(NO) #ftp数据传输端口。Default: NO
ftp_data_port=20 #默认是20，可以修改此处指定主动模式的端口
port_promiscuous=NO #默认值为NO。为YES时，取消PORT安全检查。该检查确保外出的数据只能连接到客户端上。小心打开此选项。
```

**被动模式配置**

```bash
pasv_enable=YES #YES为被动模式工作，NO则是主动模式，默认YES。
#pasv_addr_resolve=YES #是否允许vsftpd去欺骗客户。
#pasv_address=47.75.169.225 #指定被动模式回传的服务器IP地址，常用于外网访问提示返回了不可路由的服务器地址，配合pasv_addr_resolve使用。
pasv_min_port=50020 #被动模式下，数据传输端口范围，0为任意无限制。
pasv_max_port=50050 #被动模式下，数据传输端口范围，0为任意无限制。
pasv_promiscuous=YES #关闭PASV模式的安全检查，常用于解决内网访问正常，但是外网无法访问。
```

**匿名用户配置**

```bash
anonymous_enable=YES(NO) #是否允许匿名用户登入FTP，其他相关设置必须在此设置为YES时才会生效。Default: YES
anon_world_readable_only=YES(NO) #匿名用户只能下载可读。Default: YES
anon_other_write_enable=YES(NO) #是否允许匿名用户除了写入之外的权限。包括删除、修改服务器上的文件及文件名等。Default: NO
anon_mkdir_write_enable=YES(NO) #匿名用户是否可以建立目录，如果为YES，anon_other_write_enable必须为YES。Default: NO
anon_upload_enable=YES(NO) #匿名用户是否可以上传数据，如果为YES，anon_other_write_enable必须为YES。Default: NO
deny_email_enable=YES(NO) #匿名用户使用Email登录时，可以设置某此Email无法登入，与banned_email_file一起使用。Default: NO
banned_email_file=/etc/vsftpd/banned_emails #配合6，输入email地址。Default: /etc/vsftpd/banned_emails
no_anon_password=YES(NO) #当设置为YES时，匿名用户将会略过Email检验。Default: NO
anon_max_rate=0 #匿名用户的传输速度，单位bytes/second，0为不限制。Default: 0 (unlimited)
anon_umask=077 #匿名用户上传的文件的权限，如果是077，上传文件的权限是-rw-------。Default: 077
chown_uploads=NO  # 如果设置为YES，所有的匿名用户上传的文件属主都设置为chown_username指定用户
chown_username=root # 指定chown_uploads中所需的文件属主用户
deny_email_enable=NO # 如果设置为YES，则提供一个内容为mail address的文件，若是使用匿名登入，则会要求输入email address，如果它不在文件内，则禁止进入
```

**本地用户相关参数**

```bash
local_enable=YES(NO) #值为YES时，/etc/passwd内的账号才能以本地用户的方式登录FTP服务器。Default: NO
local_max_rate=0 #本地用户的传输速度限制，单位为bytes/second，0为不限制。Default: 0 (unlimited)
local_umask=022      # 本地用户的文件umask码
local_root=none      # 本地用户登陆后改变的目录，默认是各自家目录
write_enable=YES     # 本地用户写权限
```

**目录权限，锁定家目录**

```bash
chroot_local_user=YES(NO) #是否将用户限制在特定目录之内。Default: NO
chroot_list_enable=YES(NO) #是否启用chroot写入列表的功能。Default: NO
chroot_list_file=/etc/vsftpd/chroot_list #如果chroot_list_enable=YES，此文件生效Default: /etc/vsftpd.chroot_list
```

**用户访问控制**

```bash
userlist_enable=YES(NO) #阻止用户登录。Default: NO
userlist_deny=YES(NO) #当userlist_enable=YES时才会生效，或值为YES，则当使用者账号被列入userlist_file时，无法登入FTP。Default: YES
userlist_file=/etc/vsftpd/user_list #阻止登录FTP的用户列表。Default: /etc/vsftpd/user_list
```

**虚拟用户相关参数**

```bash
guest_enable=YES(NO) #值为YES时，任何用户都被映射是guest。Default: NO
guest_username=ftp #guest_enable=YES时生效，指定访客身份。Default: ftp
pam_service_name=vsftpd      # 设置PAM使用的名称
virtual_use_local_privs=NO   # 如果启用，虚拟用户将使用与本地用户相同的权限，否则与匿名用户权限相同
user_config_dir=/etc/vsftpd/virtual_user_conf/  # 指定虚拟用户配置文件目录，目录下可以创建与虚拟用户名相同的文件，给予不同的权限设置
```

**登录信息提示**

```bash
ftpd_banner=说明文字 #连接FTP时，在客户端软件上显示的说明文字。可以使用banner_file设定值来取代。Default: (none - default vsftpd banner is displayed)
banner_file=/path/file        # 设置一个包含banner信息的文件路径，开启它将会覆盖ftpd_banner
dirmessage_enable=YES(NO) #显示用户进入的目录要注意的内容，可以通过message_file修改。Default: NO
message_file=.message #当dirmessage_enable=YES时可以让vsftpd显示该档案讯息。Default: .message
```

**数据传输模式**

```bash
ascii_download_enable=YES(NO) #设置为YES，客户端优先使用ASCII格式下载文件。Default: NO
ascii_upload_enable=YES(NO) #设置为YES，客户端优先使用ASCII格式上传文件。Default: NO
```

**超时时间设置**

```bash
connect_timeout=60 #单位是秒，主动式联机模式下，发出连接信号60秒内没有响应则断线。Default: 60
1accept_timeout=60 #被动式联机模式下，服务器启用端口等待60无回应则断线。Default: 60
data_connection_timeout=300 #成功建立连接，300秒后无数据传输，客户端联机会被vsftpd强制剔除。Default: 300
idle_session_timeout=300 #300秒内无命令动作，强制脱机。Default: 300
```

**日志参数**

```bash
xferlog_enable=YES(NO) #设置为YES，会记录上传下载日志到xferlog_file设置的文件中。Default: NO (but the sample config file enables it)
xferlog_file=/var/log/xferlog #如果xferlog_enable为YES，记录上传下载日志。Default: /var/log/xferlog
xferlog_std_format=YES(NO) #是否设置为wu-ftp的登录格式。Default: NO
dual_log_enable=YES(NO) #除了/var/log/xferlog的wu-ftp格式登录日志，还可以设置具有vsftpd的独特登录日志格式。Default: NO
vsftpd_log_file=/var/log/vsftpd.log #具有vsftpd的独特登录日志格式。Default: /var/log/vsftpd.log
```

**传输速率，字节/秒**

```bash
anon_max_rate=10M 	#设置匿名登入者使用的最大传输速度，单位为B/s，0表示不限制速度。默认值为0。 
local_max_rate=10M 	#本地用户使用的最大传输速度，单位为B/s，0表示不限制速度。预设值为0。 
```



**其他设置**

```bash
download_enable=YES   # 如果设置为NO，所有下载请求被拒绝
ls_recurse_enable=NO  # 启用后，允许使用ls -R命令，此命令可能会消耗大量资源
use_localtime=YES(NO) #是否使用本地时间。Default: NO
max_clients=0 #如果vsftpd以standalone方式启动，设定同一时间内的最大连接数。Default: 0 (unlimited)
max_per_ip=0 #如果vsftpd以standalone方式启动，设定同一IP在同一时间内的最大连接数。Default: 0 (unlimited)
nopriv_user=nobody #vsftpd预设以nobody为服务执行者的权限。Default: nobody
pam_service_name=vsftpd #pam模块的名称，放置在/etc/pam.d/vsftpd中。Default: ftp
one_process_model=YES(NO) #设置为YES，每个连接都会拥有一个Process。Default: NO
tcp_wrappers=YES(NO) #支持TCP Wrappers的防火墙机制，利用/etc/hosts.[allow|deny]作为基础防火墙。Default: NO
```

英文说明：

```bash
       #anon_mkdir_write_enable
       If set to YES, anonymous users will be permitted to  create  new
       directories  under  certain  conditions. For  this to work, the
       option write_enable must be activated,  and  the anonymous  ftp
       user must have write permission on the parent directory.
       Default: NO
       #anon_other_write_enable
       If  set  to  YES,  anonymous  users will be permitted to perform
       write operations other than upload and create directory, such as
       deletion and  renaming. This  is generally not recommended but
       included for completeness.
       Default: NO
       #anon_upload_enable
       If set to YES, anonymous users will be permitted to upload files
       under   certain conditions.  For  this to  work,  the option
       write_enable must be activated, and the anonymous ftp user  must
       have write permission on desired upload locations.
       Default: NO
       #anon_world_readable_only
       When  enabled,  anonymous users will only be allowed to download
       files which are world readable. This is recognising that the ftp
       user may own files, especially in the presence of uploads.
       Default: YES
       #anonymous_enable
       Controls whether  anonymous  logins  are  permitted  or not. If
       enabled, both the usernames ftp and anonymous are recognised  as
       anonymous logins.
       Default: YES
       #ascii_download_enable
       When  enabled,  ASCII  mode  data  transfers will be honoured on
       downloads.
       Default: NO
       #ascii_upload_enable
       When enabled, ASCII mode data  transfers will  be  honoured  on
       uploads.
       Default: NO
       #async_abor_enable
       When  enabled,  a special FTP command known as "async ABOR" will
       be enabled.  Only ill advised FTP clients will use this feature.
       Additionally,  this  feature is awkward to handle, so it is dis-
       abled by default. Unfortunately, some FTP clients will hang when
       cancelling  a  transfer unless this feature is available, so you
       may wish to enable it.
       Default: NO
       #background
       When enabled, and vsftpd is started  in  "listen"  mode, vsftpd
       will  background the listener process. i.e. control will immedi-
       ately be returned to the shell which launched vsftpd.
       Default: NO
       #check_shell
       Note! This option only has  an  effect  for  non-PAM  builds  of
       vsftpd.  If  disabled,  vsftpd  will not check /etc/shells for a
       valid user shell for local logins.
       Default: YES
       #chmod_enable
       When enables, allows use of the SITE CHMOD command.  NOTE!  This
       only  applies  to  local users. Anonymous users never get to use
       SITE CHMOD.
       Default: YES
       #chown_uploads
       If enabled, all anonymously uploaded files will have the owner-
       ship  changed  to  the user specified in the setting chown_user-
       name.  This is useful from an administrative, and perhaps  secu-
       rity, standpoint.
       Default: NO
       #chroot_list_enable
       If  activated,  you  may provide  a list of local users who are
       placed in a chroot() jail in their home  directory  upon login.
       The meaning is slightly different if chroot_local_user is set to
       YES. In this case, the list becomes a list of  users  which  are
       NOT  to be placed in a chroot() jail.  By default, the file con-
       taining  this  list  is  /etc/vsftpd.chroot_list,  but  you  may
       override this with the chroot_list_file setting.
       Default: NO
       #chroot_local_user
       If  set  to  YES,  local users will be (by default) placed in a
       chroot() jail in their home  directory  after  login.   Warning:
       This  option  has security implications, especially if the users
       have upload permission, or shell access. Only enable if you know
       what  you  are doing.  Note that these security implications are
       not vsftpd specific. They apply to all FTP daemons  which  offer
       to put local users in chroot() jails.
       Default: NO
       #connect_from_port_20
       This  controls  whether  PORT style data connections use port 20
       (ftp-data) on the server machine.  For  security reasons,  some
       clients  may insist that this is the case. Conversely, disabling
       this option enables vsftpd to run with slightly less  privilege.
       Default: NO (but the sample config file enables it)
       #deny_email_enable
       If  activated,  you  may provide a list of anonymous password e-
       mail responses which cause login to be denied. By  default,  the
       file  containing this list is /etc/vsftpd.banned_emails, but you
       may override this with the banned_email_file setting.
       Default: NO
       #dirlist_enable
       If set to NO, all directory list commands will  give  permission
       denied.
       Default: YES
       #dirmessage_enable
       If  enabled,  users of the FTP server can be shown messages when
       they first enter a new directory. By  default,  a  directory  is
       scanned  for  the file .message, but that may be overridden with
       the configuration setting message_file.
       Default: NO (but the sample config file enables it)
       #download_enable
       If set to NO, all download requests will give permission denied.
       Default: YES
       #dual_log_enable
       If  enabled,  two  log files are generated in parallel, going by
       default to /var/log/xferlog and /var/log/vsftpd.log.  The former
       is  a  wu-ftpd  style transfer log, parseable by standard tools.
       The latter is vsftpd's own style log.
       Default: NO
       #force_dot_files
       If activated, files and directories  starting  with  .  will  be
       shown in directory listings even if the "a" flag was not used by
       the client. This override excludes the "." and ".." entries.
       Default: NO
       #guest_enable
       If enabled, all non-anonymous  logins  are  classed  as  "guest"
       logins.  A  guest login is remapped to the user specified in the
       guest_username setting.
       Default: NO
       #hide_ids
       If enabled, all user and group information in directory listings
       will be displayed as "ftp".
       Default: NO
       #listen 
       If  enabled, vsftpd will run in standalone mode. This means that
       vsftpd must not be run from an inetd of some kind. Instead,  the
       vsftpd  executable is run once directly. vsftpd itself will then
       take care of listening for and handling incoming connections.
       Default: NO
       #listen_ipv6
       Like the listen parameter, except vsftpd will listen on an  IPv6
       socket  instead  of  an  IPv4 one. This parameter and the listen
       parameter are mutually exclusive.
       Default: NO
       #local_enable
       Controls whether local logins are permitted or not. If  enabled,
       normal user accounts in /etc/passwd may be used to log in.
       Default: NO
       #log_ftp_protocol
       When enabled, all FTP requests and responses are logged, provid-
       ing the option xferlog_std_format is  not  enabled.  Useful  for
       debugging.
       Default: NO
       #ls_recurse_enable
       When  enabled,  this setting will allow the use of "ls -R". This
       is a minor security risk, because a ls -R at the top level of  a
       large site may consume a lot of resources.
       Default: NO
       #no_anon_password
       When  enabled, this prevents vsftpd from asking for an anonymous
       password - the anonymous user will log straight in.
       Default: NO
       #one_process_model
       If you have a Linux 2.4 kernel, it is possible to use a  differ-
       ent  security  model which only uses one process per connection.
       It is a less pure security model, but gains you performance. You
       really  don't  want  to enable this unless you know what you are
       doing, and your site supports  huge  numbers  of simultaneously
       connected users.
       Default: NO
       #passwd_chroot_enable
       If  enabled, along with chroot_local_user , then a chroot() jail
       location may be specified on a per-user basis. Each user's  jail
       is  derived from their home directory string in /etc/passwd. The
       occurrence of /./ in the home directory string denotes that  the
       jail is at that particular location in the path.
       Default: NO
       pasv_enable
       Set to NO if you want to disallow the PASV method of obtaining a
       data connection.
       Default: YES
       #pasv_promiscuous
       Set to YES if you want to disable the PASV security  check  that
       ensures  the data connection originates from the same IP address
       as the control connection.  Only enable if you know what you are
       doing!  The  only  legitimate  use  for  this is in some form of
       secure tunnelling scheme, or perhaps to facilitate FXP  support.
       Default: NO
       port_enable
       Set to NO if you want to disallow the PORT method of obtaining a
       data connection.
       Default: YES
       #port_promiscuous
       Set to YES if you want to disable the PORT security  check  that
       ensures  that  outgoing data connections can only connect to the
       client. Only enable if you know what you are doing!
       Default: NO
       #secure_email_list_enable
       Set to YES if you want only a specified list of e-mail passwords
       for  anonymous  logins  to be accepted. This is useful as a low-
       hassle way of restricting access to low-security content without
       needing  virtual users. When enabled, anonymous logins are pre-
       vented unless the password provided is listed in the file speci-
       fied  by the email_password_file setting. The file format is one
       password per line, no extra whitespace. The default filename  is
       /etc/vsftpd.email_passwords.
       Default: NO
       #session_support
       This  controls  whether vsftpd attempts to maintain sessions for
       logins. If vsftpd is  maintaining  sessions,  it will  try  and
       update  utmp  and wtmp. It will also open a pam_session if using
       PAM to authenticate, and only close this upon  logout.  You  may
       wish to disable this if you do not need session logging, and you
       wish to give vsftpd more opportunity to run with less  processes
       and  /  or  less privilege. NOTE - utmp and wtmp support is only
       provided with PAM enabled builds.
       Default: YES
       #setproctitle_enable
       If enabled, vsftpd will try and show session status  information
       in the system process listing. In other words, the reported name
       of the process will change to reflect what a vsftpd  session  is
       doing  (idle,  downloading etc). You probably want to leave this
       off for security purposes.
       Default: NO
       #syslog_enable
       If enabled, then any  log  output  which  would have  gone  to
       /var/log/vsftpd.log  goes  to the system log instead. Logging is
       done under the FTPD facility.
       Default: NO
       #tcp_wrappers
       If enabled, and vsftpd was compiled with tcp_wrappers  support,
       incoming connections  will  be  fed through tcp_wrappers access
       control. Furthermore, there is a mechanism for per-IP based con-
       figuration.  If  tcp_wrappers sets the VSFTPD_LOAD_CONF environ-
       ment variable, then the vsftpd session will  try and  load  the
       vsftpd configuration file specified in this variable.
       Default: NO
       #text_userdb_names
       By  default,  numeric IDs are shown in the user and group fields
       of directory listings. You can get  textual  names  by  enabling
       this parameter. It is off by default for performance reasons.
       Default: NO
       #use_localtime
       If  enabled, vsftpd will display directory listings with the the
       time in your local time zone. The default is to display GMT. The
       times returned by the MDTM FTP command are also affected by this
       option.
       Default: NO
       #use_sendfile
       An internal setting used for testing  the  relative  benefit  of
       using the sendfile() system call on your platform.
       Default: YES
       #userlist_deny
       This  option is examined if userlist_enable is activated. If you
       set this setting to NO, then users will be denied  login unless
       they   are   explicitly listed  in   the  file  specified  by
       userlist_file.  When login  is  denied,  the  denial  is issued
       before the user is asked for a password.
       Default: YES
       #userlist_enable
       If enabled, vsftpd will load a list of usernames, from the file-
       name given by userlist_file.  If a user tries to log in using  a
       name in this file, they will be denied before they are asked for
       a password. This may be useful in preventing cleartext passwords
       being transmitted. See also userlist_deny.
       Default: NO
       #virtual_use_local_privs
       If  enabled, virtual users will use the same privileges as local
       users. By default, virtual users will use the same privileges as
       anonymous  users, which tends to be more restrictive (especially
       in terms of write access).
       Default: NO
       #write_enable
       This controls whether any FTP commands which change the filesys-
       tem  are allowed  or not. These commands are: STOR, DELE, RNFR,
       RNTO, MKD, RMD, APPE and SITE.
       Default: NO
       #xferlog_enable
       If enabled, a log file will be maintained detailling uploads and
       downloads.    By  default,   this   file   will be  placed  at
       /var/log/vsftpd.log, but this location may be  overridden  using
       the configuration setting vsftpd_log_file.
       Default: NO (but the sample config file enables it)
       #xferlog_std_format
       If  enabled,  the  transfer log file will be written in standard
       xferlog format, as used by wu-ftpd. This is useful  because  you
       can  reuse  existing transfer statistics generators. The default
       format is more readable, however. The default location for  this
       style  of  log  file  is /var/log/xferlog, but you may change it
       with the setting xferlog_file.
       Default: NO

NUMERIC OPTIONS
       Below is a list of numeric options. A numeric option must be set  to  a
       non  negative  integer. Octal numbers are supported, for convenience of
       the umask options. To specify an octal number, use 0 as the first digit
       of the number.

       #accept_timeout
       The  timeout,  in seconds, for a remote client to establish con-
       nection with a PASV style data connection.
       Default: 60
       #anon_max_rate
       The maximum data transfer rate permitted, in bytes  per  second,
       for anonymous clients.
       Default: 0 (unlimited)
       #anon_umask
       The  value that the umask for file creation is set to for anony-
       mous users. NOTE! If you want to specify octal values,  remember
       the  "0" prefix otherwise the value will be treated as a base 10
       integer!
       Default: 077
       #connect_timeout
       The timeout, in seconds, for a remote client to respond  to  our
       PORT style data connection.
       Default: 60
       #data_connection_timeout
       The  timeout,  in  seconds, which is roughly the maximum time we
       permit data transfers to stall for  with no  progress.  If  the
       timeout triggers, the remote client is kicked off.
       Default: 300
       #file_open_mode
       The  permissions with  which uploaded files are created. Umasks
       are applied on top of this value. You may wish to change to 0777
       if you want uploaded files to be executable.
       Default: 0666
       #ftp_data_port
       The port from which PORT style connections originate (as long as
       the poorly named connect_from_port_20 is enabled).
       Default: 20
       #idle_session_timeout
       The timeout, in seconds, which is  the  maximum  time  a remote
       client  may spend between FTP commands. If the timeout triggers,
       the remote client is kicked off.
       Default: 300
       #listen_port
       If vsftpd is in standalone mode, this is the port it will listen
       on for incoming FTP connections.
       Default: 21
       #local_max_rate
       The  maximum  data transfer rate permitted, in bytes per second,
       for local authenticated users.
       Default: 0 (unlimited)
       #local_umask
       The value that the umask for file creation is set to  for  local
       users.  NOTE!  If you want to specify octal values, remember the
       "0" prefix otherwise the value will be  treated  as  a  base  10
       integer!
       Default: 077
       #max_clients
       If  vsftpd  is in standalone mode, this is the maximum number of
       clients which may be connected. Any additional clients  connect-
       ing will get an error message.
       Default: 0 (unlimited)
       #max_per_ip
       If  vsftpd  is in standalone mode, this is the maximum number of
       clients which may be connected from  the same  source  internet
       address. A client will get an error message if they go over this
       limit.
       Default: 0 (unlimited)
       #pasv_max_port
       The maximum port to allocate for PASV  style  data  connections.
       Can  be  used  to  specify  a  narrow port range to assist fire-
       walling.
       Default: 0 (use any port)
       #pasv_min_port
       The minimum port to allocate for PASV  style  data  connections.
       Can  be  used  to  specify  a  narrow port range to assist fire-
       walling.
       Default: 0 (use any port)
       #trans_chunk_size
       You probably don't want to change this, but try  setting it  to
       something like 8192 for a much smoother bandwidth limiter.
       Default: 0 (let vsftpd pick a sensible setting)

STRING OPTIONS
       Below is a list of string options.

       #anon_root
       This  option  represents a  directory  which vsftpd will try to
       change into  after  an  anonymous  login.  Failure  is  silently
       ignored.
       Default: (none)
       #banned_email_file
       This option is the name of a file containing a list of anonymous
       e-mail passwords which are not permitted. This file is consulted
       if the option deny_email_enable is enabled.
       Default: /etc/vsftpd.banned_emails
       #banner_file
       This  option  is the  name of a file containing text to display
       when someone connects to the server. If set,  it overrides  the
       banner string provided by the ftpd_banner option.
       Default: (none)
       #chown_username
       This  is the  name of the user who is given ownership of anony-
       mously uploaded files. This option is only relevant  if  another
       option, chown_uploads, is set.
       Default: root
       #chroot_list_file
       The  option  is  the  name  of a file containing a list of local
       users which will be placed in a  chroot()  jail  in  their  home
       directory.   This   option   is  only  relevant  if  the option
       chroot_list_enable is enabled. If the  option  chroot_local_user
       is  enabled,  then  the list file becomes a list of users to NOT
       place in a chroot() jail.
       Default: /etc/vsftpd.chroot_list
       #cmds_allowed
       This options specifies a comma separated list  of  allowed  FTP
       commands (post  login.  USER,  PASS and QUIT are always allowed
       pre-login). Other commands are  rejected.  This  is  a  powerful
       method   of   really   locking  down  an FTP  server.  Example:
       cmds_allowed=PASV,RETR,QUIT
       Default: (none)
       #deny_file
       This option can be used to set  a  pattern  for  filenames  (and
       directory names etc.) which should not be accessible in any way.
       The affected items are not hidden, but any attempt  to  do  any-
       thing to them (download, change into directory, affect something
       within directory etc.) will be denied. This option is very  sim-
       ple,  and  should  not  be used for serious access control - the
       filesystem's permissions should be used in preference.  However,
       this  option  may  be  useful in certain virtual user setups. In
       particular aware that if a filename is accessible by  a  variety
       of  names  (perhaps  due to symbolic links or hard links), then
       care must be taken to deny access to all the names.  Access will
       be  denied  to  items if their name contains the string given by
       hide_file, or if they match the regular expression specified  by
       hide_file.   Note that vsftpd's regular expression matching code
       is a simple implementation which is a  subset  of  full  regular
       expression  functionality.  Because  of  this,  you will need to
       carefully and exhaustively test any application of this  option.
       And  you are  recommended to use filesystem permissions for any
       important security policies due to  their  greater  reliability.
       Example: deny_file={*.mp3,*.mov,.private}
       Default: (none)
       #email_password_file
       This  option  can be used to provide an alternate file for usage
       by the secure_email_list_enable setting.
       Default: /etc/vsftpd.email_passwords
       #ftp_username
       This is the name of the user we use for handling anonymous  FTP.
       The home directory of this user is the root of the anonymous FTP
       area.
       Default: ftp
      # ftpd_banner
       This string option allows you to override  the  greeting banner
       displayed by vsftpd when a connection first comes in.
       Default: (none - default vsftpd banner is displayed)
       guest_username
       See  the boolean setting guest_enable for a description of what
       constitutes a guest login. This setting  is  the real  username
       which guest users are mapped to.
       Default: ftp
       #hide_file
       This  option  can  be  used  to set a pattern for filenames (and
       directory names etc.) which  should  be  hidden  from  directory
       listings. Despite being hidden, the files / directories etc. are
       fully accessible to clients who know what names to actually use.
       Items  will be hidden if their names contain the string given by
       hide_file, or if they match the regular expression specified  by
       hide_file.  Note that vsftpd's regular expression matching code
       is a simple implementation which is a  subset  of  full  regular
       expression   functionality.    Example: hide_file={*.mp3,.hid-
       den,hide*,h?}
       Default: (none)
       #listen_address
       If vsftpd is in standalone mode, the default listen address  (of
       all local interfaces) may be overridden by this setting. Provide
       a numeric IP address.
       Default: (none)
       #listen_address6
       Like listen_address, but specifies a default listen address  for
       the  IPv6 listener (which is used if listen_ipv6 is set). Format
       is standard IPv6 address format.
       Default: (none)
       #local_root
       This option represents a directory  which  vsftpd  will  try  to
       change into after a local (i.e. non-anonymous) login. Failure is
       silently ignored.
       Default: (none)
       #message_file
       This option is the name of the file  we  look  for  when a  new
       directory  is  entered. The contents are displayed to the remote
       user.   This   option   is   only   relevant   if   the option
       dirmessage_enable is enabled.
       Default: .message
       #nopriv_user
       This is the name of the user that is used by vsftpd when it want
       to be totally unprivileged. Note that this should be a dedicated
       user,  rather  than nobody. The user nobody tends to be used for
       rather a lot of important things on most machines.
       Default: nobody
       pam_service_name
       This string is the name of the PAM service vsftpd will use.
       Default: ftp
       #pasv_address
       Use this option to override the  IP  address  that  vsftpd  will
       advertise  in response to the PASV command. Provide a numeric IP
       address.
       Default: (none - the address is taken  from  the incoming  con-
       nected socket)
       #secure_chroot_dir
       This  option  should  be the name of a directory which is empty.
       Also, the directory should not be writable by the ftp user. This
       directory is used as a secure chroot() jail at times vsftpd does
       not require filesystem access.
       Default: /usr/share/empty
       #user_config_dir
       This powerful option allows the override of  any config option
       specified in the manual page, on a per-user basis. Usage is sim-
       ple, and is  best  illustrated  with  an example.  If  you  set
       user_config_dir  to  be /etc/vsftpd_user_conf and then log on as
       the user "chris", then vsftpd will apply the  settings  in  the
       file  /etc/vsftpd_user_conf/chris  for  the duration of the ses-
       sion. The format of this file is as  detailed  in  this manual
       page!  PLEASE NOTE that not all settings are effective on a per-
       user basis. For example, many settings only prior to the user's
       session  being  started. Examples  of  settings which will not
       affect any behviour on a per-user basis include  listen_address,
       banner_file, max_per_ip, max_clients, xferlog_file, etc.
       Default: (none)
       #user_sub_token
       This  option  is useful is conjunction with virtual users. It is
       used to automatically generate a home directory for each virtual
       user, based on a template. For example, if the home directory of
       the  real  user  specified  via  guest_username  is   /home/vir-
       tual/$USER,  and user_sub_token is set to $USER, then when vir-
       tual user fred logs in, he will end up (usually chroot()'ed)  in
       the directory /home/virtual/fred.  This option also takes affect
       if local_root contains user_sub_token.
       Default: (none)
       #userlist_file
       This  option  is the  name  of  the  file   loaded   when   the
       userlist_enable option is active.
       Default: /etc/vsftpd.user_list
       #vsftpd_log_file
       This option is the name of the file to which we write the vsftpd
       style  log  file.  This  log  is only  written  if  the option
       xferlog_enable is set, and xferlog_std_format is NOT set. Alter-
       natively,  it  is  written  if   you   have   set   the option
       dual_log_enable.  One  further  complication  - if you have set
       syslog_enable, then this file is not written and output is  sent
       to the system log instead.
       Default: /var/log/vsftpd.log
       #xferlog_file
       This  option  is the name of the file to which we write the wu-
       ftpd style transfer log. The transfer log is only written if the
       option  xferlog_enable  is  set, along with xferlog_std_format.
       Alternatively,  it  is  written  if  you have  set  the option
       dual_log_enable.
       Default: /var/log/xferlog
```





**测试访问：**

**验证结果：匿名用户权限：默认只可以下载**

Windows资源管理器输入：ftp://10.0.0.21/

![1567064195437](assets/1567064195437.png)

测试上传的权限：

![1567064263921](assets/1567064263921.png)

测试删除权限：

![1567064311632](assets/1567064311632.png)



# 4.4 使用SSL为FTP加密

**查看当前vsftpd是否具有ssl模块**

```bash
[root@localhost ~]# ldd $(which vsftpd) |grep ssl
    libssl.so.10 => /usr/lib64/libssl.so.10 (0x00007f55009bf000)
```

**利用OpenSSL生成证书**

```bash
[root@localhost] openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout /etc/vsftpd/vsftpd.pem -out /etc/vsftpd/vsftpd.pem
       其中"-days 365"声明证书的有效期是一年。
       接下来的过程需要你输入一些相关的国家，地区，位置，组织名称，common name等信息。
       回答这些信息以后系统会将生成完的证书vsftpd.pem文件保存在/etc/vsftpd目录下。

```

**查看证书**

```
[root@localhost ~]# openssl x509 -in /etc/vsftpd/vsftpd.pem -noout -text
```

**配置vsftp支持ssl**

**注：在编辑vstpd.conf时，注意格式，选项行后不得有空格**

```bash
#启用ssl
ssl_enable=YES
#匿名不支持SSL
allow_anon_ssl=NO
#本地用户登录加密
force_local_logins_ssl=YES
#本地数据传输加密
force_local_data_ssl=YES
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO
rsa_cert_file=/etc/vsftpd/vsftpd.pem

```



# 4.5 MYSQL存储VSFTPD虚拟用户

**1、安装pam_mysql**

```bash
yum -y install pam_mysql
```

**2、创建数据库表和用户**

```bash
mysql> create database vsftpd;
 
mysql> grant select on vsftpd.* to vsftpd@'10.0.0.%' identified by '123456';
mysql> flush privileges;
```

**3、创建表结构，添加数据（虚拟用户的账号和密码）**

根据需要添加所需要的用户，需要说明的是，这里将其密码为了安全起见应该使用PASSWORD函数加密后存储。

```bash
mysql> use vsftpd;
mysql> create table users (
    -> id int AUTO_INCREMENT NOT NULL,
    -> name char(20) binary NOT NULL,
    -> password char(48) binary NOT NULL,
    -> primary key(id)
    -> ); 
    
mysql> insert into users(name,password) values('vstom',password('123456'));
mysql> insert into users(name,password) values('vsjerry',password('123456'));
```

**4、手动配置pam_mysql**
原本pam的身份验证是基于/etc/pam.d/vsftp文件，现修改为基于MySQL的vsftpd.mysql。

```bash
vim /etc/pam.d/vsftpd.mysql
auth required pam_mysql.so user=vstom passwd=123456 host=10.0.0.10 db=vsftpd table=users usercolumn=name passwdcolumn=password crypt=2
account required pam_mysql.so user=vsjerry passwd=123456 host=10.0.0.10 db=vsftpd table=users usercolumn=name passwdcolumn=password crypt=2
```

 详解：

​    user：远程连接mysql的用户名

​    password：远程连接mysql的密码

​    host：远程连接mysql的服务器地址

​    db：连接的数据库名

​    table：连接数据库中的表名

​    usercolumn：表中的字段名

​    passwdcolumn：表中密码字段的名称

​    crypt：该方法加密的密码

​    0：没有加密。密码存储在纯文本

​    1：使用crypt(3)功能

​    2：使用mysql密码

​    3：使用md5加密

​    4：使用sha加密





# 4.2 FTP登录方式

## 4.2.1 匿名用户登录

### 4.2.1.1 匿名用户常用参数

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

### 4.2.1.2 测试匿名用户登录

所在文件夹要有写入权限 例如：`chmod -R o+w /var/ftp/ `

匿名用户家目录/var/ftp权限是755，这个权限是不能改变的。

![1567065435709](assets/1567065435709.png)





## 4.2.2 本地用户登录

### 4.2.2.1 创建本地用户tom并设置密码

如果tom用户已存在则不需要创建

```shell
useradd   tom
echo '123456'|passwd --stdin tom
```

### 4.2.2.2 本地用户常用参数

```shell
local_enable=YES	#设定本地用户可以访问
write_enable=YES	#全局设置，是否容许写入（无论是匿名用户还是本地用户，若要启用上传权限的话，就要开启他）
local_umask=022 	#设定上传后文件的权限掩码
local_root=/home/tom		#本地用户ftp根目录，默认是本地用户的家目录
local_max_rate=0			#本地用户最大传输速率（字节）。0为不限制
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





### 4.2.2.3 测试本地用户上传下载

**使用winscp客户端**

![1567330500061](assets/1567330500061.png)



![1567330557462](assets/1567330557462.png)



## 4.2.3 虚拟用户登陆

当我们配置了虚拟用户登陆时，FTP会将保存在文件或数据库中虚拟出的用户，映射到指定的系统账号（/sbin/nologin)来访问资源，其中这些虚拟用户是FTP的用户

可以通过修改配置的虚拟账号文件，来控制用户的登陆，安全性更高





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







