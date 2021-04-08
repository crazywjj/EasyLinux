[TOC]





# Linux安全相关



# 1 密码错误自动锁定用户

根据当前系统使用的pam_tally版本，来操作不通的指令。

```
[root@localhost ~]# find / -name 'pam_tally2.so'
/usr/lib64/security/pam_tally2.so
```

**1.1 在字符终端下，实现某一用户连续错误登陆N次后，就锁定该用户X分钟(pam_tally2)**

此方式只限制了终端

执行 `vim /etc/pam.d/login`
在#%PAM-1.0 下新起一行，加入

```
auth required pam_tally2.so deny=5 unlock_time=600 even_deny_root root_unlock_time=1200
```

如果不限制root用户，则可以写成

```
auth required pam_tally2.so deny=5 unlock_time=600
```

- even_deny_root ：也限制root用户；
- deny ：设置普通用户和root用户连续错误登陆的最大次数，超过最大次数，则锁定该用户；
- unlock_time ：设定普通用户锁定后，多少时间后解锁，单位是秒；
- root_unlock_time ：设定root用户锁定后，多少时间后解锁，单位是秒；

备注：
1、此处使用的是 pam_tally2 模块，如果不支持 pam_tally2 模块可以使用 pam_tally 模块。另外，不同的pam版本，设置可能有所不同，具体使用方法，可以参照相关模块的使用规则。

2、也可以直接在 system-auth 文件中直接添加这些命令，修改完成后，凡是调用 system-auth 文件的服务，都会生效。因为有自动解锁时间，所以，不用担心全部限制后，会出现永远无法登陆的”尴尬”情况。

3、可以使用 pam_tally2 -r -u username 命令，手动清除某用户记录次数。

**1.2 设置Linux用户连续N次登陆失败时，自动锁定X分钟(pam_tally)**

1、如果想在所有登陆方式上，限制所有用户，可以在 `/etc/pam.d/system-auth` 中增加2行

```bash
auth required pam_tally.so onerr=fail no_magic_root
account required pam_tally.so deny=3 no_magic_root even_deny_root_account per_user reset
```

deny ：设置普通用户和root用户连续错误登陆的最大次数，超过最大次数，则锁定该用户；
no_magic_root：连root用户也在限制范围，不给root特殊权限。
详细参数的含义，参见 /usr/share/doc/pam-xxxx/txts/README.pam_tally

如果不想限制root用户，可以将 even_deny_root_account 取消掉。

2、针对不同服务来限制不同登陆方式

在远程telnet、ssh登陆上做限制，可以编辑如下文件，添加的内容和上方也一样。
vim /etc/pam.d/remote

3、手动解除锁定：

查看某一用户错误登陆次数：
pam_tally –user username

清空某一用户错误登陆次数：
pam_tally –user username –reset

faillog -r 命令亦可。

4、pam_tally没有自动解锁功能

因为pam_tally没有自动解锁的功能，只能等解锁时间到期。所以，做限制时要多加注意，如果root也被锁了，就只能够进单用户模式解锁了。当然，也可以添加crontab任务，达到定时自动解锁的功能，但需要注意的是，如果在/etc /pam.d/system-auth 文件中添加了pam_tally的话，当root被锁定后，crontab任务会失效，所以，最好不要在system-auth 文件中添加pam_tally。





# 2 设置登录超时自动断开

用户为了增强linux系统的安全性，需要在用户操作空闲一段时间后自动断开。

```bash
export TMOUT=600   #600秒无操作自动退出登录
readonly TMOUT	# 将值设置为readonly 防止用户更改，在shell中无法修改TMOUT
```

unset命令临时取消TMOUT

```
unset TMOUT
```



