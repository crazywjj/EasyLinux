[TOC]







# 第三单元-mysql数据库的高级管理



## 3.1 mysql密码的修改与恢复

### 3.1.1 修改密码

**命令行修改**

```
mysqladmin -uroot -p123456 password 654321
```

​		

**数据库内修改**
方法一：

```
update mysql.user set password=password('123456') where user='root' and host='localhost';
flush privileges;
```

方法二：

```
set password for root@'localhost' =password('654321');
#注：这种方法无需刷新权限
```

方法三：命令行执行

```
mysql_secure_installation
#安全配置向导，会对数据库进行简单的优化。
```





### 3.1.2 忘记mysql密码后的恢复

```
1、/etc/init.d/mysqld stop
2、mysqld_safe --skip-grant-tables --user=mysql &>/dev/null &
3、直接mysql无密码登录	
4、然后设置新密码
```





## 3.2 mysql备份与恢复

### 3.2.1 备份的概念与分类

**概念**

为防止文件、数据丢失或损坏等可能出现的意外情况，将电子计算机存储设备中的数据复制到磁带等大容量存储设备中,从而在原文中独立出来单独贮存的程序或文件副本; 如果系统的硬件或存储媒体发生故障，“备份”工具可以帮助您保护数据免受意外的损失。

mysql数据备份其实就是通过SQL语句的形式将数据DUMP出来，以文件的形式保存，而且导出的文件还是可编辑的，这和Oracle数据库的rman备份还是很不一样的，mysql更像是一种逻辑备份从库中抽取SQL语句，这就包括建库，连库，建表，插入等就像是将我们之前的操作再通过SQL语句重做一次。



**分类**

**一般的备份可分为：**

1、系统备份：指的是用户操作系统因磁盘损伤或损坏，计算机病毒或人为误删除等原因造成的系统文件丢失，从而造成计算机操作系统不能正常引导，因此使用系统备份，将操作系统事先贮存起来，用于故障后的后备支援。

2、数据备份：指的是用户将数据包括文件，数据库，应用程序等贮存起来，用于数据恢复时使用。

**备份更专业地可分为：**

i. 全量备份：完全备份就是指对某一个时间点上的所有数据或应用进行的一个完全拷贝

ii. 增量备份：增量备份是指在一次全备份或上一次增量备份后，以后每次的备份只需备份与前一次相比增加和者被修改的文件

iii. 差异备份：差异备份是指在一次全备份后到进行差异备份的这段时间内，对那些增加或者修改文件的备份





### 3.2.2 备份工具：mysqldump、mydumper、xtrabackup



#### 3.2.2.1 使用mysqldump工具备份与恢复

**1.命令介绍：**

mysqldump是mysql提供的一个基于命令行的mysql数据备份工具，提供了丰富的参数选择，用于各种需求形式的备份，如单库备份，多库备份，单表与多表备份，全库备份，备份表结构，备份表数据等。



**2.备份还原语法格式：**

**以下为备份还原，lol数据库和hero表为演练。**

**（1）、单库备份及还原**

```
备份
mysqldump -uroot -p123456 lol >/opt/backup/lol.sql
注意：此操作只备份其中的表（包括创建表的语句和数据）。

还原
mysql -uroot -p123456 -e 'create database lol;'
mysql -uroot -p123456 lol </opt/backup/lol.sql
```

**（2）、多库备份及还原**

```
备份
mysqldump -uroot -p123456 -B 库1 库2 库3 >/opt/backup/mysql_bak_db.sql

还原
mysql -uroot -p123456 </opt/backup/mysql_bak_db.sql
```

**注意：多个库之间用空格分隔**

**（3）、单表备份及还原**

```
备份
mysqldump -uroot -p123456 lol hero>/opt/backup/hero.sql

还原
mysql -uroot -p123456 lol </opt/backup/hero.sql
```

**（4）、多表备份及还原**

```
备份
mysqldump -uroot -p123456 库名 表1 表2>/opt/backup/mysql_bak_db.sql

还原
mysql -uroot -p123456 库名 </opt/backup/mysql_bak_db.sql
```

**（5）、全库备份**

```
mysqldump -uroot -p123456 -A >/opt/backup/mysql_bak_db.sql
或者
mysqldump -uroot –p123456 --all-databases >/opt/backup/mysql_bak_db.sql
```



**3.常用参数解析：**

**（1）、-B等同于--databases**

如果想一次备份多个库需要添加B参数，B参数会在备份数据中添加create database和use语句

**（2）、-F**
在备份之前会先刷新日志，可以看到二进制文件前滚产生新的二进制文件。
**（3）、--master-data**
有二个值1或者2，等于1会在备份数据中增加如下语句：

```
CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000040',MASTER_LOG_POS=4543;
```

等于2会在备份数据中增加如下语句：

```
-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000040',MASTER_LOG_POS=4543;
```

唯一区别就是有没有被“--”注释掉，如果备份的数据用于slave，等于1即可，此时从库就知道应该从哪个地方开始读二进制日志，如果仅用于备份标识当前二进制是哪一个和位置点等于2合适。

**（4）、-x**  

锁表，备份的时候锁表来保证数据一致性。

**（5）、-d**  

只备份表结构不备份数据。

**（6）、-A等同于--all-databases**

备份所有数据库

**（7）、-X等同于--xml**

导出为xml文件

**（8）、--single-transaction** 
Innodb引擎保证数据一致性的参数，使用此参数后会话的安全隔离级别会被置为repeatble-read，此时其它会话提交的数据是不可视的，从而保证数据的一致性。
**更多参数请参考help**

```sql
--all-databases  , -A
导出全部数据库。
mysqldump  -uroot -p --all-databases
--all-tablespaces  , -Y
导出全部表空间。
mysqldump  -uroot -p --all-databases --all-tablespaces
--no-tablespaces  , -y
不导出任何表空间信息。
mysqldump  -uroot -p --all-databases --no-tablespaces
--add-drop-database
每个数据库创建之前添加drop数据库语句。
mysqldump  -uroot -p --all-databases --add-drop-database
--add-drop-table
每个数据表创建之前添加drop数据表语句。(默认为打开状态，使用--skip-add-drop-table取消选项)
mysqldump  -uroot -p --all-databases  (默认添加drop语句)
mysqldump  -uroot -p --all-databases –skip-add-drop-table  (取消drop语句)
--add-locks
在每个表导出之前增加LOCK TABLES并且之后UNLOCK  TABLE。(默认为打开状态，使用--skip-add-locks取消选项)
mysqldump  -uroot -p --all-databases  (默认添加LOCK语句)
mysqldump  -uroot -p --all-databases –skip-add-locks   (取消LOCK语句)
--allow-keywords
允许创建是关键词的列名字。这由表名前缀于每个列名做到。
mysqldump  -uroot -p --all-databases --allow-keywords
--apply-slave-statements
在'CHANGE MASTER'前添加'STOP SLAVE'，并且在导出的最后添加'START SLAVE'。
mysqldump  -uroot -p --all-databases --apply-slave-statements
--character-sets-dir
字符集文件的目录
mysqldump  -uroot -p --all-databases  --character-sets-dir=/usr/local/mysql/share/mysql/charsets
--comments
附加注释信息。默认为打开，可以用--skip-comments取消
mysqldump  -uroot -p --all-databases  (默认记录注释)
mysqldump  -uroot -p --all-databases --skip-comments   (取消注释)
--compatible
导出的数据将和其它数据库或旧版本的MySQL 相兼容。值可以为ansi、mysql323、mysql40、postgresql、oracle、mssql、db2、maxdb、no_key_options、no_tables_options、no_field_options等，
要使用几个值，用逗号将它们隔开。它并不保证能完全兼容，而是尽量兼容。
mysqldump  -uroot -p --all-databases --compatible=ansi
--compact
导出更少的输出信息(用于调试)。去掉注释和头尾等结构。可以使用选项：--skip-add-drop-table  --skip-add-locks --skip-comments --skip-disable-keys
mysqldump  -uroot -p --all-databases --compact
--complete-insert,  -c
使用完整的insert语句(包含列名称)。这么做能提高插入效率，但是可能会受到max_allowed_packet参数的影响而导致插入失败。
mysqldump  -uroot -p --all-databases --complete-insert
--compress, -C
在客户端和服务器之间启用压缩传递所有信息
mysqldump  -uroot -p --all-databases --compress
--create-options,  -a
在CREATE TABLE语句中包括所有MySQL特性选项。(默认为打开状态)
mysqldump  -uroot -p --all-databases
--databases,  -B
导出几个数据库。参数后面所有名字参量都被看作数据库名。
mysqldump  -uroot -p --databases test mysql
--debug
输出debug信息，用于调试。默认值为：d:t,/tmp/mysqldump.trace
mysqldump  -uroot -p --all-databases --debug
mysqldump  -uroot -p --all-databases --debug=” d:t,/tmp/debug.trace”
--debug-check
检查内存和打开文件使用说明并退出。
mysqldump  -uroot -p --all-databases --debug-check
--debug-info
输出调试信息并退出
mysqldump  -uroot -p --all-databases --debug-info
--default-character-set
设置默认字符集，默认值为utf8
mysqldump  -uroot -p --all-databases --default-character-set=utf8
--delayed-insert
采用延时插入方式（INSERT DELAYED）导出数据
mysqldump  -uroot -p --all-databases --delayed-insert
--delete-master-logs
master备份后删除日志. 这个参数将自动激活--master-data。
mysqldump  -uroot -p --all-databases --delete-master-logs
--disable-keys
对于每个表，用/*!40000 ALTER TABLE tbl_name DISABLE KEYS */;和/*!40000 ALTER TABLE tbl_name ENABLE KEYS */;语句引用INSERT语句。这样可以更快地导入dump出来的文件，因为它是在插入所有行后创建索引的。该选项只适合MyISAM表，默认为打开状态。
mysqldump  -uroot -p --all-databases 
--dump-slave
该选项将主的binlog位置和文件名追加到导出数据的文件中(show slave status)。设置为1时，将会以CHANGE MASTER命令输出到数据文件；设置为2时，会在change前加上注释。该选项将会打开--lock-all-tables，除非--single-transaction被指定。该选项会自动关闭--lock-tables选项。默认值为0。
mysqldump  -uroot -p --all-databases --dump-slave=1
mysqldump  -uroot -p --all-databases --dump-slave=2
--master-data
该选项将当前服务器的binlog的位置和文件名追加到输出文件中(show master status)。如果为1，将会输出CHANGE MASTER 命令；如果为2，输出的CHANGE  MASTER命令前添加注释信息。该选项将打开--lock-all-tables 选项，除非--single-transaction也被指定（在这种情况下，全局读锁在开始导出时获得很短的时间；其他内容参考下面的--single-transaction选项）。该选项自动关闭--lock-tables选项。
mysqldump  -uroot -p --host=localhost --all-databases --master-data=1;
mysqldump  -uroot -p --host=localhost --all-databases --master-data=2;
--events, -E
导出事件。
mysqldump  -uroot -p --all-databases --events
--extended-insert,  -e
使用具有多个VALUES列的INSERT语法。这样使导出文件更小，并加速导入时的速度。默认为打开状态，使用--skip-extended-insert取消选项。
mysqldump  -uroot -p --all-databases
mysqldump  -uroot -p --all-databases--skip-extended-insert   (取消选项)
--fields-terminated-by
导出文件中忽略给定字段。与--tab选项一起使用，不能用于--databases和--all-databases选项
mysqldump  -uroot -p test test --tab=”/home/mysql” --fields-terminated-by=”#”
--fields-enclosed-by
输出文件中的各个字段用给定字符包裹。与--tab选项一起使用，不能用于--databases和--all-databases选项
mysqldump  -uroot -p test test --tab=”/home/mysql” --fields-enclosed-by=”#”
--fields-optionally-enclosed-by
输出文件中的各个字段用给定字符选择性包裹。与--tab选项一起使用，不能用于--databases和--all-databases选项
mysqldump  -uroot -p test test --tab=”/home/mysql”  --fields-enclosed-by=”#” --fields-optionally-enclosed-by  =”#”
--fields-escaped-by
输出文件中的各个字段忽略给定字符。与--tab选项一起使用，不能用于--databases和--all-databases选项
mysqldump  -uroot -p mysql user --tab=”/home/mysql” --fields-escaped-by=”#”
--flush-logs
开始导出之前刷新日志。
请注意：假如一次导出多个数据库(使用选项--databases或者--all-databases)，将会逐个数据库刷新日志。除使用--lock-all-tables或者--master-data外。在这种情况下，日志将会被刷新一次，相应的所以表同时被锁定。因此，如果打算同时导出和刷新日志应该使用--lock-all-tables 或者--master-data 和--flush-logs。
mysqldump  -uroot -p --all-databases --flush-logs
--flush-privileges
在导出mysql数据库之后，发出一条FLUSH  PRIVILEGES 语句。为了正确恢复，该选项应该用于导出mysql数据库和依赖mysql数据库数据的任何时候。
mysqldump  -uroot -p --all-databases --flush-privileges
--force
在导出过程中忽略出现的SQL错误。
mysqldump  -uroot -p --all-databases --force
--help
显示帮助信息并退出。
mysqldump  --help
--hex-blob
使用十六进制格式导出二进制字符串字段。如果有二进制数据就必须使用该选项。影响到的字段类型有BINARY、VARBINARY、BLOB。
mysqldump  -uroot -p --all-databases --hex-blob
--host, -h
需要导出的主机信息
mysqldump  -uroot -p --host=localhost --all-databases
--ignore-table
不导出指定表。指定忽略多个表时，需要重复多次，每次一个表。每个表必须同时指定数据库和表名。例如：--ignore-table=database.table1 --ignore-table=database.table2 ……
mysqldump  -uroot -p --host=localhost --all-databases --ignore-table=mysql.user
--include-master-host-port
在--dump-slave产生的'CHANGE  MASTER TO..'语句中增加'MASTER_HOST=<host>，MASTER_PORT=<port>'  
mysqldump  -uroot -p --host=localhost --all-databases --include-master-host-port
--insert-ignore
在插入行时使用INSERT IGNORE语句.
mysqldump  -uroot -p --host=localhost --all-databases --insert-ignore
--lines-terminated-by
输出文件的每行用给定字符串划分。与--tab选项一起使用，不能用于--databases和--all-databases选项。
mysqldump  -uroot -p --host=localhost test test --tab=”/tmp/mysql”  --lines-terminated-by=”##”
--lock-all-tables,  -x
提交请求锁定所有数据库中的所有表，以保证数据的一致性。这是一个全局读锁，并且自动关闭--single-transaction 和--lock-tables 选项。
mysqldump  -uroot -p --host=localhost --all-databases --lock-all-tables
--lock-tables,  -l
开始导出前，锁定所有表。用READ  LOCAL锁定表以允许MyISAM表并行插入。对于支持事务的表例如InnoDB和BDB，--single-transaction是一个更好的选择，因为它根本不需要锁定表。
请注意当导出多个数据库时，--lock-tables分别为每个数据库锁定表。因此，该选项不能保证导出文件中的表在数据库之间的逻辑一致性。不同数据库表的导出状态可以完全不同。
mysqldump  -uroot -p --host=localhost --all-databases --lock-tables
--log-error
附加警告和错误信息到给定文件
mysqldump  -uroot -p --host=localhost --all-databases  --log-error=/tmp/mysqldump_error_log.err
--max_allowed_packet
服务器发送和接受的最大包长度。
mysqldump  -uroot -p --host=localhost --all-databases --max_allowed_packet=10240
--net_buffer_length
TCP/IP和socket连接的缓存大小。
mysqldump  -uroot -p --host=localhost --all-databases --net_buffer_length=1024
--no-autocommit
使用autocommit/commit 语句包裹表。
mysqldump  -uroot -p --host=localhost --all-databases --no-autocommit
--no-create-db,  -n
只导出数据，而不添加CREATE DATABASE 语句。
mysqldump  -uroot -p --host=localhost --all-databases --no-create-db
--no-create-info,  -t
只导出数据，而不添加CREATE TABLE 语句。
mysqldump  -uroot -p --host=localhost --all-databases --no-create-info
--no-data, -d
不导出任何数据，只导出数据库表结构。
mysqldump  -uroot -p --host=localhost --all-databases --no-data
--no-set-names,  -N
等同于--skip-set-charset
mysqldump  -uroot -p --host=localhost --all-databases --no-set-names
--opt
等同于--add-drop-table,  --add-locks, --create-options, --quick, --extended-insert, --lock-tables,  --set-charset, --disable-keys 该选项默认开启,  可以用--skip-opt禁用.
mysqldump  -uroot -p --host=localhost --all-databases --opt
--order-by-primary
如果存在主键，或者第一个唯一键，对每个表的记录进行排序。在导出MyISAM表到InnoDB表时有效，但会使得导出工作花费很长时间。 
mysqldump  -uroot -p --host=localhost --all-databases --order-by-primary
--password, -p
连接数据库密码
--pipe(windows系统可用)
使用命名管道连接mysql
mysqldump  -uroot -p --host=localhost --all-databases --pipe
--port, -P
连接数据库端口号
--protocol
使用的连接协议，包括：tcp, socket, pipe, memory.
mysqldump  -uroot -p --host=localhost --all-databases --protocol=tcp
--quick, -q
不缓冲查询，直接导出到标准输出。默认为打开状态，使用--skip-quick取消该选项。
mysqldump  -uroot -p --host=localhost --all-databases 
mysqldump  -uroot -p --host=localhost --all-databases --skip-quick
--quote-names,-Q
使用（`）引起表和列名。默认为打开状态，使用--skip-quote-names取消该选项。
mysqldump  -uroot -p --host=localhost --all-databases
mysqldump  -uroot -p --host=localhost --all-databases --skip-quote-names
--replace
使用REPLACE INTO 取代INSERT INTO.
mysqldump  -uroot -p --host=localhost --all-databases --replace
--result-file,  -r
直接输出到指定文件中。该选项应该用在使用回车换行对（\\r\\n）换行的系统上（例如：DOS，Windows）。该选项确保只有一行被使用。
mysqldump  -uroot -p --host=localhost --all-databases --result-file=/tmp/mysqldump_result_file.txt
--routines, -R
导出存储过程以及自定义函数。
mysqldump  -uroot -p --host=localhost --all-databases --routines
--set-charset
添加'SET NAMES  default_character_set'到输出文件。默认为打开状态，使用--skip-set-charset关闭选项。
mysqldump  -uroot -p --host=localhost --all-databases 
mysqldump  -uroot -p --host=localhost --all-databases --skip-set-charset
--single-transaction
该选项在导出数据之前提交一个BEGIN SQL语句，BEGIN 不会阻塞任何应用程序且能保证导出时数据库的一致性状态。它只适用于多版本存储引擎，仅InnoDB。本选项和--lock-tables 选项是互斥的，因为LOCK  TABLES 会使任何挂起的事务隐含提交。要想导出大表的话，应结合使用--quick 选项。
mysqldump  -uroot -p --host=localhost --all-databases --single-transaction
--dump-date
将导出时间添加到输出文件中。默认为打开状态，使用--skip-dump-date关闭选项。
mysqldump  -uroot -p --host=localhost --all-databases
mysqldump  -uroot -p --host=localhost --all-databases --skip-dump-date
--skip-opt
禁用–opt选项.
mysqldump  -uroot -p --host=localhost --all-databases --skip-opt
--socket,-S
指定连接mysql的socket文件位置，默认路径/tmp/mysql.sock
mysqldump  -uroot -p --host=localhost --all-databases --socket=/tmp/mysqld.sock
--tab,-T
为每个表在给定路径创建tab分割的文本文件。注意：仅仅用于mysqldump和mysqld服务器运行在相同机器上。注意使用--tab不能指定--databases参数
mysqldump  -uroot -p --host=localhost test test --tab="/home/mysql"
--tables
覆盖--databases (-B)参数，指定需要导出的表名，在后面的版本会使用table取代tables。
mysqldump  -uroot -p --host=localhost --databases test --tables test
--triggers
导出触发器。该选项默认启用，用--skip-triggers禁用它。
mysqldump  -uroot -p --host=localhost --all-databases --triggers
--tz-utc
在导出顶部设置时区TIME_ZONE='+00:00' ，以保证在不同时区导出的TIMESTAMP 数据或者数据被移动其他时区时的正确性。
mysqldump  -uroot -p --host=localhost --all-databases --tz-utc
--user, -u
指定连接的用户名。
--verbose, --v
输出多种平台信息。
--version, -V
输出mysqldump版本信息并退出
--where, -w
只转储给定的WHERE条件选择的记录。请注意如果条件包含命令解释符专用空格或字符，一定要将条件引用起来。
mysqldump  -uroot -p --host=localhost --all-databases --where=” user=’root’”
--xml, -X
导出XML格式.
mysqldump  -uroot -p --host=localhost --all-databases --xml
--plugin_dir
客户端插件的目录，用于兼容不同的插件版本。
mysqldump  -uroot -p --host=localhost --all-databases --plugin_dir=”/usr/local/lib/plugin”
--default_auth
客户端插件默认使用权限。
mysqldump  -uroot -p --host=localhost --all-databases --default-auth=”/usr/local/lib/plugin/<PLUGIN>”
```





**4.还原备份**

方法一：

```mysql
#备份lol数据库
[root@ c6s02 ~]# mysqldump -uroot -p123456 -B lol >lol.sql


#先删除lol数据库
[root@ c6s02 ~]# mysql -uroot -p123456

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| lol                |
| mysql              |
| performance_schema |
| test               |
| wg                 |
+--------------------+
6 rows in set (0.00 sec)

mysql> drop database lol;
Query OK, 3 rows affected (0.03 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
| wg                 |
+--------------------+
5 rows in set (0.00 sec)

mysql> \q

#测试恢复并查看
[root@ c6s02 ~]# mysql -uroot -p123456 <lol.sql
Warning: Using a password on the command line interface can be insecure.

[root@ c6s02 ~]# mysql -uroot -p123456
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| lol                |
| mysql              |
| performance_schema |
| test               |
| wg                 |
+--------------------+
6 rows in set (0.00 sec)

mysql> select * from lol.hero;
+----+--------+--------+-----------+--------+
| id | 角色   | 职业   | 攻击力    | 血量   |
+----+--------+--------+-----------+--------+
|  1 | 蛮王   | 战士   |       200 | NULL   |
|  2 | 狗头   | 战士   |       100 | NULL   |
|  3 | 剑圣   | 战士   |       300 | NULL   |
+----+--------+--------+-----------+--------+
3 rows in set (0.00 sec)
```

方法二：source方法

```
[root@ c6s02 ~]# mysql -uroot -p123456
mysql> source /root/lol.sql
```











#### 3.2.2.2 Mydumper工具介绍与使用

**1.介绍**
mydumper是针对mysql数据库备份的一个轻量级第三方的开源工具，备份方式为逻辑备份。它支持多线程，备份速度远高于原生态的mysqldump以及众多优异特性。因此该工具是DBA们的不二选择。



**2.mydumper的特点**

- 轻量级C语言写的
- 执行速度比mysqldump快10倍，多线程逻辑备份,生产的多个备份文件
- 事务性和非事务性表一致的快照(适用于0.2.2以上版本)
- 支持文件压缩，支持导出binlog，支持多线程恢复，支持将备份文件切块
- 多线程恢复(适用于0.2.1以上版本)
- 以守护进程的工作方式，定时快照和连续二进制日志(适用于0.5.0以上版本)
- 保证备份数据的一致性,
- 与mysqldump相同，备份时对 MyISAM 表施加FTWRL (FLUSH TABLES WITH READ LOCK), 会阻塞DML 语句



**3.mydumper安装**

此项目托管在github。

github地址：https://github.com/maxbube/mydumper

安装

```shell
rpm安装
wget https://github.com/maxbube/mydumper/releases/download/v0.9.5/mydumper-0.9.5-2.el7.x86_64.rpm
rpm -ivh mydumper-0.9.5-2.el7.x86_64.rpm

或者
源码安装
yum -y install glib2-devel mysql-devel zlib-devel pcre-devel cmake gcc-c++ git
git clone https://github.com/maxbube/mydumper.git
cd mydumper
cmake .
make && make install


#测试安装可能会有以下报错：
报错一：
[root@ c6s02 mydumper]# mydumper -V
mydumper: error while loading shared libraries: libmysqlclient.so.18: cannot open shared object file: No such file or directory

#解决办法：
[root@ c6s02 ~]# find / -name libmysqlclient.so.18.1.0
/usr/local/mysql/lib/libmysqlclient.so.18.1.0	--#找到自己mysql数据库下的libmysqlclient.so.18.1.0并做软连接
/root/mysql-5.6.45/libmysql/libmysqlclient.so.18.1.0

#做软连接
[root@ c6s02 ~]# ln -sv  /usr/local/mysql/lib/libmysqlclient.so.18.1.0 /lib64/libmysqlclient.so.18

报错二：
mydumper: error while loading shared libraries: libpcre.so.1: cannot open shared object file: No such file or directory

wget https://ftp.pcre.org/pub/pcre/pcre-8.00.tar.gz
tar -zxvf  pcre-8.00.tar.gz
cd pcre-8.00
./configure --enable-utf8
make
make check
make install

#显示以下效果表示安装成功
[root@ c6s02 ~]# mydumper -V
mydumper 0.10.0, built against MySQL 5.6.45
```



**4.mydumper语法及参数介绍**

```
mydumper -u [USER] -p [PASSWORD] -h [HOST] -P [PORT] -t [THREADS] -b -c -B [DB] -o [directory]
```

注意：命令行之间要有空格 -u 用户名  -p 密码 之间必须有空格

```
-B, --database 需要备份的库
-T, --tables-list 需要备份的表，用，分隔
-o, --outputdir 输出文件的目录
-s, --statement-size Attempted size of INSERT statement in bytes, default 1000000
-r, --rows 试图分裂成很多行块表
-c, --compress 压缩输出文件
-e, --build-empty-files 即使表没有数据，还是产生一个空文件
-x, --regex 支持正则表达式
-i, --ignore-engines 忽略的存储引擎，用，分隔
-m, --no-schemas 不导出表结构
-k, --no-locks 不执行临时共享读锁 警告：这将导致不一致的备份
-l, --long-query-guard 长查询，默认60s
--kill-long-queries kill掉长时间执行的查询(instead of aborting)
-b, --binlogs 导出binlog
-D, --daemon 启用守护进程模式
-I, --snapshot-interval dump快照间隔时间，默认60s，需要在daemon模式下
-L, --logfile 日志文件
-h, --host
-u, --user
-p, --password
-P, --port
-S, --socket
-t, --threads 使用的线程数，默认4
-C, --compress-protocol 在mysql连接上使用压缩
-V, --version
 -v, --verbose 更多输出, 0 = silent, 1 = errors, 2 = warnings, 3 = info, default 2
```



**5.myloader参数介绍：**

```
-d, --directory 导入备份目录
-q, --queries-per-transaction 每次执行的查询数量, 默认1000
-o, --overwrite-tables 如果表存在删除表
-B, --database 需要还原的库
-e, --enable-binlog 启用二进制恢复数据
-h, --host
-u, --user
-p, --password
-P, --port
-S, --socket
-t, --threads 使用的线程数量，默认4
-C, --compress-protocol 连接上使用压缩
-V, --version
-v, --verbose 更多输出, 0 = silent, 1 = errors, 2 = warnings, 3 = info, default 2
```



**6.mydumper输出文件：**

```
metadata:元数据 记录备份开始和结束时间，以及binlog日志文件位置。
table data:每个表一个文件
table schemas:表结构文件
binary logs: 启用--binlogs选项后，二进制文件存放在binlog_snapshot目录下
daemon mode:在这个模式下，有五个目录0，1，binlogs，binlog_snapshot，last_dump。
备份目录是0和1，间隔备份，如果mydumper因某种原因失败而仍然有一个好的快照，
当快照完成后，last_dump指向该备份。
```

**7.常用备份示例：**

备份单个库 

```
mydumper -u 用户名 -p 密码 -B 需要备份的库名 -o /tmp/bak

-B,需要备份的库   -o 输出文件的目录（备份输出指定的目录）
```

备份所有数据库

```
全库备份期间除了information_schema与performance_schema之外的库都会被备份

mydumper -u 用户名 -p 密码 -o /tmp/bak
 
 -o 输出文件的目录（备份输出指定的目录）
```

备份单表

```
mydumper -u 用户名 -p 密码 -B 库名 -T 表名 -o /tmp/bak

-T 需要备份的表，多表用逗号分隔 -o指定输出备份文件路径 
```

备份多表

```
mydumper -u 用户名 -p 密码 -B 库名 -T 表1,表2 -o /tmp/bak

当前目录自动生成备份日期时间文件夹,不指定-o参数及值时默认为：export-20150703-145806

mydumper -u 用户名 -p 密码 -B 数据库名字 -T 表名
```

不带表结构备份表

```
mydumper -u 用户名 -p 密码 -B 数据名字 -T 表名 -m

-m 不导出表结构
```

压缩备份及连接使用压缩协议(非本地备份时)

```
mydumper -u 用户名 -p 密码 -B 数据库名字 -o /tmp/bak -c -C

-c  压缩输出文件 -C 在mysql连接上使用压缩协议  -o 输出文件的目录（备份输出指定的目录）
```

备份特定表

```
mydumper -u 用户名 -p 密码 -B 数据库名字  --regex=actor* -o /tmp/bak

只备份以actor*开头的表

-x 正则表达式: 'db.table'  --regex  
```

过滤特定库，如本来不备份mysql及test库

```
mydumper -u 用户名 -p 密码 -B 数据库名字 --regex '^(?!(mysql|test))' -o /tmp/bak
```

基于空表产生表结构文件

```
mydumper -u 用户名 -p 密码 -B 数据库名字 -T 空表 -e -o /tmp/bak

-e 即使表没有数据，还是产生一个空文件 
```

设置长查询的上限，如果存在比这个还长的查询则退出mydumper，也可以设置杀掉这个长查询

```
mydumper -u root -p pwd -B sakila --long-query-guard 200 --kill-long-queries
```

备份时输出详细更多日志

```
mydumper -u 用户名 -p 密码 -B 数据库名字 -T 空表 -v 3 -o /tmp/bak

-v 更多输出, 0 = silent, 1 = errors, 2 = warnings, 3 = info,详细输出 default 2
```

导出binlog，使用-b参数，会自动在导出目录生成binlog_snapshot文件夹及binlog

```
mydumper -u root -p pwd -P 3306 -b -o /tmp/bak
```

总结：
mysql备份，备份数据库、备份数据表。恢复也是恢复数据库，恢复数据表。







#### 3.2.2.3 xtrabackup

课外阅读：Xtrabackup工具介绍与使用

a) 了解Xtrabackup的介绍与使用举例

Xtrabackup是由percona提供的mysql数据库备份工具, 有两个主要的工具：xtrabackup、innobackupex，备份过程快速、可靠，备份过程不会打断正在执行的事务，能够基于压缩等功能节约磁盘空间和流量，自动实现备份检验，还原速度快，备份可在线备份，但是恢复要关闭服务器，恢复后再启动.



### 3.3 mysql大数据库的备份思路

小量的数据库我们可以每天进行完整备份，因为这也用不了多少时间。但当数据库很大时，我们就不太可能每天进行一次完整备份了，而且改成每周一次完整备份，每天一次增量备份类似这样的备份策略。**增量备份的原理就是使用了MySQL的二进制日志，所以我们必须启用二进制日志功能。**

mysqldump   数据量<=30G

xtrabackup   数据量>=30G



## 3.4 mysql的安全配置

### 3.4.1 mysql用户的操作

**查看用户:**

```
select user,host from mysql.user;
```

**创建用户：**

```
CREATE USER '用户'@'主机' IDENTIFIED BY '密码';
create user 'boy'@'locahost' identified by '123456';  #只能连接
```

**删除用户：**

```
drop user 'user'@'主机域';

**特殊的删除方法：**
delete from mysql.user where  user='bbs' and host='172.16.1.%'; 
flush privileges;
```

**创建用户同时授权：**

```
grant all on *.* to boy@'172.16.1.%' identified by '123456';
flush privileges;
```



### 3.4.2 mysql用户的权限设置

权限可以分为四个层级：全局级别（*.*）、数据库级别(数据库名.*)、表级别(数据库名.表名)、列级别(  权限（列）    数据库名.表名)。

全局级别的权限存放在mysql.user表中，数据库级别的权限存放在mysql.db或者mysql.host，表级别的权限存放在mysql.tables_priv中，列级别的权限存放在mysql.columns_priv中。

**查看用户对应的权限：**

```
show grants for 用户@主机域\G
show grants for root@localhost\G
```

**给用户授权：**

```
GRANT ALL ON 数据库.表 TO '用户'@'localhost';
GRANT ALL ON db1.* TO 'jeffrey'@'localhost';
GRANT ALL ON *.* TO 'boy'@'localhost';
```

**收回权限：**

```
REVOKE INSERT ON *.* FROM boy@localhost;
```

**可以授权的用户权限：**

```
INSERT,SELECT, UPDATE, DELETE, CREATE, DROP, RELOAD, SHUTDOWN, PROCESS, FILE, REFERENCES, INDEX, ALTER, SHOW DATABASES, SUPER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER, CREATE TABLESPACE
```



## 3.5 mysql日志类型

### 3.5.1 二进制日志（log-bin）

二进制日志非常重要，二进制日志会记录mysql数据库的所有变更操作，其实和oracle的redolog日志原理差不多，由于它记录的所有的操作，于是我们就可以使用某个时间点之后的二进制日志做前滚操作，来增量恢复数据。

mysql的二进制日志可以使用mysqlbinlog来进行查看和过滤，一直过滤到我们想要的数据再导入数据库，而且也是非常方便的，但尤其要强调的是要严格按照二进制日志生成的顺序执行。

**用途：**记录所有变更操作，用于增量备份

**配置：**在my.cnf中添加

```
[mysqld]
log-bin =mysql-bin
log-bin-index =mysql-bin.index
```



### 3.5.2 中继日志（relay-log）

顾名思义，传递日志，主要用在主从复制的架构中，只在从库中有中继日志（多级复制除外）在从库中将主库复制过来的二进制日志保存为中继日志，**用于从库重构数据**。

**配置：**在my.cnf中添加

```
[mysqld]
relay-log =relay-log
relay_log_index= relay-log.index
```



### 3.5.3 慢查询日志（slow_query_log）

慢查询日志主要用于mysql优化，从数据库中找出哪些SQL语句是比较慢的，将其放到一个文件中，后续可以使用mysqlsla工具去对慢查询语句进行分析，将分析结果提交给开发进行SQL优化。

**用途：**找出慢查询进行优化

**配置：**在my.cnf中添加

```
[mysqld]
slow_query_log= 1
long-query-time= 2  
slow_query_log_file= /data/3306/slow.log
```



### 3.5.4 一般查询日志

会记录所有访问mysql的行业，因此会产生大量日志，一般建议关闭。

配置：在my.cnf中添加

```
[mysqld]
general_log = 1
log_output =FILE
general_log_file= /home/mysql/mysql/log/mysql.log 
```



### 3.5.5 错误日志

记录mysql产生的错误，这个日志在排错的时候相当有用，一般建议开启。

配置：在my.cnf中添加

```
[mysqld]
log-warnings =1
log-error =/home/mysql/mysql/log/mysql.err
```



### 3.5.6 事务日志

缓存事务提交的数据，实现将随机IO转换成顺序IO。

配置：在my.cnf中添加

```
[mysqld]
innodb_log_buffer_size= 16M
innodb_log_file_size= 128M
innodb_log_files_in_group= 3
```

