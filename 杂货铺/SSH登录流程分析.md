[TOC]







# SSH登录流程分析

# SSH介绍

SSH（Secure Shell）是一套协议标准，可以用来实现两台机器之间的安全登录以及安全的数据传送，其保证数据安全的原理是**非对称加密**。

SSH 为 Secure Shell 的缩写，由 IETF 的网络小组（Network Working Group）所制定；SSH 为建立在应用层基础上的安全协议。SSH 是目前较可靠，专为远程登录会话和其他网络服务提供安全性的协议。利用 SSH 协议可以有效防止远程管理过程中的信息泄露问题。

# SSH工作原理

SSH的安全性比较好，其对数据进行加密的方式主要有两种：对称加密（密钥加密）和非对称加密（公钥加密）。

对称加密指加密解密使用的是同一套秘钥。Client端把密钥加密后发送给Server端，Server用同一套密钥解密。对称加密的加密强度比较高，很难破解。但是，Client数量庞大，很难保证密钥不泄漏。如果有一个Client端的密钥泄漏，那么整个系统的安全性就存在严重的漏洞。为了解决对称加密的漏洞，于是就产生了非对称加密。非对称加密有两个密钥：“公钥”和“私钥”。公钥加密后的密文，只能通过对应的私钥进行解密。想从公钥推理出私钥几乎不可能，所以非对称加密的安全性比较高。



# 中间人攻击

SSH之所以能够保证安全，原因在于它采用了公钥加密，这个过程本身是安全的，整个过程是这样的：（1）远程主机收到用户的登录请求，把自己的公钥发给用户。（2）用户使用这个公钥，将登录密码加密后，发送回来。（3）远程主机用自己的私钥，解密登录密码，如果密码正确，就同意用户登录。

但是实际用的时候存在一个风险：如果有人截获了登录请求，然后冒充远程主机，将伪造的公钥发给用户，那么用户很难辨别真伪。因为不像https协议，SSH协议的公钥是没有证书中心（CA）公证的，是自己签发的。

如果攻击者插在用户与远程主机之间（比如在公共的wifi区域），用伪造的公钥，获取用户的登录密码。再用这个密码登录远程主机，那么SSH的安全机制就不存在了。这种风险就是著名的”中间人攻击”（Man-in-the-middle attack）。那么SSH协议是怎样应对的呢？known_hosts文件可以有效预防



# 口令登录

如果是第一次登录远程机，会出现以下提示：

```bash
$ ssh user@host
The authenticity of host 'host (12.18.429.21)' can't be established.
RSA key fingerprint is 98:2e:d7:e0:de:9f:ac:67:28:c2:42:2d:37:16:58:4d.
Are you sure you want to continue connecting (yes/no)?
```

因为公钥长度较长（采用RSA算法，长达1024位），很难比对，所以对其进行MD5计算，将它变成一个128位的指纹。如98:2e:d7:e0:de:9f:ac:67:28:c2:42:2d:37:16:58:4d，这样比对就容易多了。

经过比对后，如果用户接受这个远程主机的公钥，系统会出现一句提示语：

```sh
Warning: Permanently added 'host,12.18.429.21' (RSA) to the list of known hosts.
```

表示host主机已得到认可，然后再输入登录密码就可以登录了。

当远程主机的公钥被接受以后，它就会被保存在文件~/.ssh/known_hosts之中。下次再连接这台主机，系统就会认出它的公钥已经保存在本地了，从而跳过警告部分，直接提示输入密码。每个SSH用户都有自己的known_hosts文件，此外系统也有一个这样的文件，一般是/etc/ssh/ssh_known_hosts，保存一些对所有用户都可信赖的远程主机的公钥。



# 公钥登录

使用密码登录，每次都必须输入密码，非常麻烦。好在SSH还提供了公钥登录，可以省去输入密码的步骤。 所谓”公钥登录”，原理很简单，就是用户将自己的公钥储存在远程主机上。登录的时候，远程主机会向用户发送一段随机字符串，用户用自己的私钥加密后，再发回来。远程主机用事先储存的公钥进行解密，如果成功，就证明用户是可信的，直接允许登录shell，不再要求密码。

这种方法要求用户必须提供自己的公钥。如果没有现成的，可以直接用ssh-keygen生成一个： 

```sh
$ ssh-keygen
```

运行上面的命令以后，系统会出现一系列提示，可以一路回车。其中有一个问题是，要不要对私钥设置口令（passphrase），如果担心私钥的安全，这里可以设置一个。 运行结束以后，在~/.ssh/目录下，会新生成两个文件：id_rsa.pub和id_rsa。前者是公钥，后者是私钥。

这时再输入下面的命令，将公钥传送到远程主机host上面：

```sh
$ ssh-copy-id user@host
```

远程主机将用户的公钥，保存在登录后的用户主目录的~/.ssh/authorized_keys文件中。 这样，以后就登录远程主机不需要输入密码了。

如果还是不行，就用vim打开远程主机的/etc/ssh/sshd_config这个文件，将以下几行的注释去掉。

```sh
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```

然后，重启远程主机的ssh服务。



# SSH登录解析

密钥登陆比密码登陆安全，主要是由于他使用了非对称加密，登陆过程当中须要用到**密钥对**。整个登陆流程以下：

1. 远程服务器持有公钥（id_rsa.pub），当有用户进行登陆，服务器就会随机生成一串字符串，而后发送给正在进行登陆的用户。
2. 用户收到远程服务器发来的字符串，使用与**远程服务器公钥配对的私钥(id_rsa)**对字符串进行加密，再发送给远程服务器。
3. 服务器使用公钥对用户发来的加密字符串进行解密，获得的解密字符串若是与第一步中发送给客户端的随机字符串同样，那么判断为登陆成功。

## 生成密钥对

使用 `ssh-keygen` 就能够直接生成登陆须要的密钥对。`ssh-keygen` 是 Linux 下的命令，不添加任何参数就能够生成密钥对。

```sh
[root@localhost ~]# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):(1)
Enter passphrase (empty for no passphrase):(2)
Enter same passphrase again:(3)
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:Eq35CQt9dAdEOc8K2D77u0UFcsn1+JbxFV06sPg6w8c root@localhost.localdomain

```

执行 `ssh-keygen` 会出现如上的提示，在 `(1)` 处这里提示用户输入生成的私钥的名称，若是不填，默认私钥保存在 `/root/.ssh/id_rsa` 文件中。这里要注意两点：

- 生成的密钥，会放在**执行 `ssh-keygen` 命令的用户的家目录**下的 `.ssh` 文件夹中。即 `$HOME/.ssh/` 目录下。
- 生成的公钥的文件名，一般是私钥的文件名后面加 `.pub` 的后缀。

`(2)` 处，提示输入密码，注意这里的密码是用来保证私钥的安全的。若是填写了密码，那么在使用密钥进行登陆的时候，会让你输入密码，这样子保证了若是私钥丢失了不至于被恶意使用。

`(3)` 是重复 `(2)` 输入的密码。

生成的私钥还要注意一点：**私钥的权限应该为 `rw-------`，若是私钥的权限过大，那么私钥任何人均可以读写就会变得不安全。ssh 登陆就会失败。**



## 复制公钥到客户端

ssh-copy-id其实就是把本机的公钥，复制到远程机的`/root/.ssh/authorized_keys`文件。

```sh
[root@localhost ~]# ssh-copy-id -i /root/.ssh/id_rsa.pub root@10.159.238.11
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host '10.159.238.11 (10.159.238.11)' can't be established.
ECDSA key fingerprint is SHA256:rVYlxedRWbuNoaGmSidJYQxdO1YHoigVxgcpnLXFbnI.
ECDSA key fingerprint is MD5:8c:01:99:86:4a:88:90:c9:3e:ce:b0:fd:2e:4a:57:7b.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@10.159.238.11's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@10.159.238.11'"
and check to make sure that only the key(s) you wanted were added.

# 成功后本机会增加生成known_hosts文件，
[root@localhost ~]# ll /root/.ssh/
total 12
-rw------- 1 root root 1675 Jun 16 10:16 id_rsa
-rw-r--r-- 1 root root  408 Jun 16 10:16 id_rsa.pub
-rw-r--r-- 1 root root  175 Jun 16 10:27 known_hosts

[root@localhost ~]# cat /root/.ssh/known_hosts
10.159.238.11 ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBGhqXbVPnKzr6gKGL99+3aRao+wTuUP5hejK/vYivHjDHzBXBfnwyxXX4HZ0SyEfeWjTDdQneekBRcx/N2oWzaY=

```

`i`: 指定公钥

常见文件：

- id_rsa：保存私钥
- id_rsa.pub：保存公钥
- authorized_keys：保存已授权的客户端公钥
- known_hosts：保存已认证的远程主机ID（关于known_hosts详情，见文末更新内容）



## 免密登录远程机

指定非默认端口号连接：

```sh
ssh -p 5522 username@hostname
```

使用私钥连接到远程主机：

```sh
ssh -i /root/.ssh/id_rsa username@hostname
```



# known_hosts文件作用

known_hosts文件是SSH客户端中的一个重要配置文件。当首次与一个SSH服务器建立连接时，客户端会记录下该服务器返回的的公钥，并保存在known_hosts文件中，以后每次连接该服务器时，客户端都会验证该服务器返回的公钥是否与known_hosts文件中保存的一致。如果不一致，则会发出警告，提示可能存在DNS劫持、中间人攻击等安全问题。**因此，known_hosts文件可以保证SSH连接的安全性，防止恶意攻击。**



# 常见问题

## SSH的警告信息

在SSH中也用到了证书，可以使用ECDSA或者RSA方式结合摘要技术生成表示服务器身份的密钥指纹。而一旦当此指纹发生变化时，ssh则会提示有可能有实际的风险，可能会提示如下信息：

```sh
[root@host121 .ssh]# ssh 192.168.163.121
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:y9ir2Jbc7kNZPhP9h/O9juUZbTmGDo6NZi2IZnLwg0s.
Please contact your system administrator.
Add correct host key in /root/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /root/.ssh/known_hosts:2
ECDSA host key for 192.168.163.121 has changed and you have requested strict checking.
Host key verification failed.

```

注意提示信息的主要内容：

- REMOTE HOST IDENTIFICATION HAS CHANGED： 远程连接的身份发生了变化，因为中间人攻击的方式最重要的一个特点就是服务器侧和客户端没有意识到这个中间人角色的存在，所以进行提示是很重要的，这里提示你之前连接的这台主机已经发生变化了。
- IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!： 大写一般可表示警告，告诉使用者可能有人在做很讨厌的事情，比如MITM。
- Someone could be eavesdropping on you right now (man-in-the-middle attack)!：提示可能有人在监控你。
  

产生SSH的MITM警告信息的条件和方法：

只要事前存在的know_hosts中保存的连接对象机器的密钥指纹一旦发生变化，ssh的时候就会提示上述信息，但是在实际情况中，很多时候此密钥指纹都会发生变化，比如：

1. 远程服务器重装或更换了系统，导致系统生成的密钥发生了变化。
2. 本地计算机重装了系统或者更改了SSH客户端软件。
3. 发生了中间人攻击，远程服务器可能是伪造的。



如果确定是第三种情况，即中间人攻击，此时请不要继续连接，并做好安全防护措施。如果确定连接是安全的，可以通过如下几种方法解决:

1.运行如下命令，刷新known_hosts中对应远程服务器公钥，推荐此方法

```sh
ssh-keygen -R server_ip_address
ssh-keyscan -H server_ip_address >> ~/.ssh/known_hosts
```

2.直接删除known_hosts文件

```sh
rm -f ~/.ssh/known_hosts
```

或者只删除对应ip的相关公钥信息

编辑 ~/.ssh/known_hosts 文件，将目标ip公钥信息删除后保存即可。

3.关闭known_hosts验证

修改配置文件，在ssh登陆时不通过known_hosts文件进行验证（安全性有所降低），修改完需重启机器

```sh
vi ~/.ssh/config      //编辑配置文件
# 添加以下两行代码：
StrictHostKeyChecking no 
UserKnownHostsFile /dev/null
```





























































