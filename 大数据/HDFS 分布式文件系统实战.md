

# HDP环境使用



# HDFS 分布式文件系统实战

// 查看命令的帮助文档

```
[hdfs@node1 ~]$ hdfs dfs -help
```



// 显示目录信息

```
[hdfs@node1 ~]$ hdfs dfs -ls /
Found 12 items
drwxrwxrwt   - yarn   hadoop          0 2024-01-16 12:57 /app-logs
drwxr-xr-x   - hdfs   hdfs            0 2024-01-16 12:53 /apps
drwxr-xr-x   - yarn   hadoop          0 2024-01-16 12:53 /ats
drwxr-xr-x   - hdfs   hdfs            0 2024-01-16 12:53 /atsv2
drwxr-xr-x   - hdfs   hdfs            0 2024-01-16 12:53 /hdp
drwx------   - livy   hdfs            0 2024-01-16 12:56 /livy2-recovery
drwxr-xr-x   - mapred hdfs            0 2024-01-16 12:53 /mapred
drwxrwxrwx   - mapred hadoop          0 2024-01-16 12:53 /mr-history
drwxrwxrwx   - spark  hadoop          0 2024-01-16 14:30 /spark2-history
drwxrwxrwx   - hdfs   hdfs            0 2024-01-16 12:58 /tmp
drwxr-xr-x   - hdfs   hdfs            0 2024-01-16 12:57 /user
drwxr-xr-x   - hdfs   hdfs            0 2024-01-16 12:54 /warehouse
```



// 在 hdfs 上创建目录,需要使用对应的管理员用户（即hdfs）才有相应权限

```
[hdfs@node1 ~]$ hdfs dfs -mkdir /user/stu01
[hdfs@node1 ~]$ hdfs dfs -ls /user
Found 6 items
drwxrwx---   - ambari-qa hdfs          0 2024-01-16 12:57 /user/ambari-qa
drwxr-xr-x   - hbase     hdfs          0 2024-01-16 12:53 /user/hbase
drwxr-xr-x   - hive      hdfs          0 2024-01-16 12:54 /user/hive
drwxrwxr-x   - livy      hdfs          0 2024-01-16 12:56 /user/livy
drwxrwxr-x   - spark     hdfs          0 2024-01-16 12:57 /user/spark
drwxr-xr-x   - hdfs      hdfs          0 2024-01-16 14:32 /user/stu01

```



// 上传 Linux 系统中的文件到 HDFS 文件系统的指定目录.

```
[hdfs@node1 ~]$ hdfs dfs -put ./osquery /user/stu01
[hdfs@node1 ~]$ hdfs dfs -ls /user/stu01
Found 1 items
drwxr-xr-x   - hdfs hdfs          0 2024-01-16 14:40 /user/stu01/osquery
[hdfs@node1 ~]$ hdfs dfs -ls /user/stu01/osquery
Found 8 items
-rw-r--r--   3 hdfs hdfs       4127 2024-01-16 14:40 /user/stu01/osquery/deploy.sh
-rw-r--r--   3 hdfs hdfs   14108945 2024-01-16 14:40 /user/stu01/osquery/grpc.ext
-rw-r--r--   3 hdfs hdfs        232 2024-01-16 14:40 /user/stu01/osquery/osquery-tls.flags
-rw-r--r--   3 hdfs hdfs         40 2024-01-16 14:40 /user/stu01/osquery/osquery.autoload
-rw-r--r--   3 hdfs hdfs        179 2024-01-16 14:40 /user/stu01/osquery/osquery.cfg
-rw-r--r--   3 hdfs hdfs      35392 2024-01-16 14:40 /user/stu01/osquery/osquery.conf
-rw-r--r--   3 hdfs hdfs        249 2024-01-16 14:40 /user/stu01/osquery/osquery.flags
-rw-r--r--   3 hdfs hdfs        114 2024-01-16 14:40 /user/stu01/osquery/osquery.logrotate.conf

```



// 显示文件内容

```
[hadoop@node1 test1]$ hdfs dfs -cat /user/stu01/stu01.txt
```



// 从hdfs的一个目录拷贝 hdfs 的另一个目录

```
[hadoop@node1 ~]$ hdfs dfs -cp /user/stu02/stu01-4.txt /user/stu01/
```



```
// 在 hdfs 目录中移动文件
[hadoop@node1 ~]$ hdfs dfs -mv /user/stu02/5 /user/stu01/
```



//等同于 copyToLocal，就是从 hdfs 下载文件到本地

```
[hadoop@node1 test1]$ hdfs dfs -get /user/stu01/5
```





// 统计文件夹的大小信息

```
[hadoop@node1 test1]$ hdfs dfs -du -s -h /user/stu01
```



// 统计一个指定目录下的文件节点数量

```
[hadoop@node1 test1]$ hdfs dfs -count -v /user/stu01
```

删除命令

```
hfds dfs -rm [-f] [-r|-R] [-skipTrash] <src> ...
注意:如果删除文件夹需要加-r

hfds dfs -rmdir [--ignore-fail-on-non-empty] <dir> ...
注意:必须是空文件夹,如果非空必须使用rm删除
```



查看磁盘利用率和文件大小

```shell
hfds dfs -df [-h] [<path> ...]]     #查看分布式系统的磁盘使用情况
hfds dfs -du [-s] [-h] <path> ...	#查看分布式系统上当前路径下文件的情况	-h：human 以人类可读的方式显示
```



修改权限
跟本地的操作一致,-R是让子目录或文件也进行相应的修改

```
hfds dfs -chgrp [-R] GROUP PATH...
hfds dfs -chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...
hfds dfs -chown [-R] [OWNER][:[GROUP]] PATH...
```

修改文件的副本数

```shell
调用格式:hadoop fs -setrep  3 /   将hdfs根目录及子目录下的内容设置成3个副本
注意:当设置的副本数量与初始化时默认的副本数量不一致时,集群会作出反应,比原来多了会自动进行复制.
```



查看文件的状态

```
命令的作用:当向hdfs上写文件时，可以通过dfs.blocksize配置项来设置文件的block的大小。这就导致了hdfs上的不同的文件block的大小是不相同的。有时候想知道hdfs上某个文件的block大小，可以预先估算一下计算的task的个数。stat的意义：可以查看文件的一些属性。

调用格式:hdfs dfs -stat [format] 文件路径
format的形式：
%b：打印文件的大小（目录大小为0）
%n：打印文件名
%o：打印block的size
%r：打印副本数
%y：utc时间 yyyy-MM-dd HH:mm:ss
%Y：打印自1970年1月1日以来的utc的微秒数
%F：目录打印directory，文件打印regular file
注意:

当使用-stat命令但不指定format时，只打印创建时间，相当于%y

-stat 后面只跟目录,%r,%o等打印的都是0,只有文件才有副本和大小
```















# Hive 数据仓库实战







https://blog.51cto.com/u_76633/5754946



