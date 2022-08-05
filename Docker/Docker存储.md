







# Docker存储

https://docs.docker.com/storage/

默认情况下，在容器内创建的所有文件都存储在可写容器层上。这意味着：

- 当该容器不再存在时，数据不会持续存在，并且如果另一个进程需要数据，则可能很难将数据从容器中取出。
- 容器的可写层与运行容器的主机紧密耦合。您无法轻松地将数据移动到其他地方。
- 写入容器的可写层需要 [存储驱动程序](https://docs.docker.com/storage/storagedriver/)来管理文件系统。存储驱动程序提供了一个联合文件系统，使用 Linux 内核。与使用直接写入主机文件系统的*数据卷*相比，这种额外的抽象会降低性能 。

Docker 有两个选项让容器在主机上存储文件，以便即使在容器停止后文件也能持久保存：*volumes*和 *bind mounts*。

Docker 还支持将文件存储在主机内存中的容器。此类文件不会持久保存。如果您在 Linux 上运行 Docker，则使用*tmpfs mount*将文件存储在主机的系统内存中。如果您在 Windows 上运行 Docker，*命名管道*用于将文件存储在主机的系统内存中。



# Docker四种存储方式

docker四种方式：默认、volumes数据卷、bind mounts挂载、tmpfs mount(仅在linux环境中提供)，其中volumes、bind mounts两种实现持久化容器数据；

- 默认：数据保存在运行的容器中，容器删除后，数据也随之删除；
- volumes：数据卷，数据存放在主机文件系统/var/lib/docker/volumes/目录下，该目录由docker管理，其它进程不允许修改，推荐该种方式持久化数据；
- Bind mounts：直接挂载主机文件系统的任何目录或文件，类似主机和容器的共享目录，主机上任何进程都可以访问修改，容器中也可以看到修改，这种方式最简单。
- tmpfs：数据暂存在主机内存中，不会写入文件系统，重启后，数据删除。

下图展示了Volumes，Bind mounts和tmpfs mounts三种存储技术的不同：

![](assets/types-of-mounts-volume.png)







1、Volumes使用场景

在多个容器间共享数据。
无法确保Docker主机一定拥有某个指定的文件夹或目录结构，使用Volumes可以屏蔽这些宿主机差异。
当你希望将数据存储在远程主机或云提供商上。
当你希望备份，恢复或者迁移数据从一台Docker主机到另一台Docker主机，Volumes是更好的选择。

2、Bind mounts使用场景

在宿主机和容器间共享配置文件。例如将nginx容器的配置文件保存在宿主机上，通过Bind mounts挂载后就不用进入容器来修改nginx的配置了。
在宿主机和容器间共享代码或者build输出。例如将宿主机某个项目的target目录挂载到容器中，这样在宿主机上Maven build出一个最新的产品，可以直接咋i容器中运行，而不用生成一个新的镜像。
Docker主机上的文件或目录结构是确定的

3、tmpfs mounts使用场景

当你因为安全或其他原因，不希望将数据持久化到容器或宿主机上，那你可以使用tmpfs mounts模式。

4、Bind mounts和Volumes行为上的差异

如果你将一个空Volume挂载到一个非空容器目录上，那么这个容器目录中的文件会被复制到Volume中，即容器目录原有文件不会被Volume覆盖。
如果你使用Bind mounts将一个宿主机目录挂载到容器目录上，此容器目录中原有的文件会被隐藏，从而只能读取到宿主机目录下的文件

**附：bind mount 与 docker managed volume 的区别：**

这两种 **data volume** 实际上都是使用 **host** 文件系统的中的某个路径作为 **mount** 源。它们不同之处在于：

| **不同点**                 | **bind mount**                  | **docker managed volume**        |
| -------------------------- | ------------------------------- | -------------------------------- |
| **volume 位置**            | 可任意指定                      | **/var/lib/docker/volumes/...**  |
| **对已有mount point 影响** | 隐藏并替换为 **volume**         | 原有数据复制到 **volume**        |
| **是否支持单个文件**       | 支持                            | 不支持，只能是目录               |
| **权限控制**               | 可设置为只读，默认为读写权限    | 无控制，均为读写权限             |
| **移植性**                 | 移植性弱，与 **host path** 绑定 | 移植性强，无需指定 **host** 目录 |



# Docker存储驱动

docker提供了多种存储驱动来实现不同的方式存储镜像，下面是常用的几种存储驱动：

- AUFS
- OverlayFS
- Devicemapper
- Btrfs
- ZFS



常用存储驱动对比

| 存储驱动     | 特点                                     | 优点                                                         | 缺点                                                         | 适用场景                           |
| ------------ | ---------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------- |
| AUFS         | 联合文件系统、未并入内核主线、文件级存储 | 作为docker的第一个存储驱动，已经有很长的历史，比较稳定，且在大量的生产中实践过，有较强的社区支持 | 有多层，在做写时复制操作时，如果文件比较大且存在比较低的层，可能会慢一些 | 大并发但少IO的场景                 |
| overlayFS    | 联合文件系统、并入内核主线、文件级存储   | 只有两层                                                     | 不管修改的内容大小都会复制整个文件，对大文件进行修改显示要比小文件消耗更多的时间 | 大并发但少IO的场景                 |
| Devicemapper | 并入内核主线、块级存储                   | 块级无论是大文件还是小文件都只复制需要修改的块，并不是整个文件 | 不支持共享存储，当有多个容器读同一个文件时，需要生成多个复本，在很多容器启停的情况下可能会导致磁盘溢出 | 适合io密集的场景                   |
| Btrfs        | 并入linux内核、文件级存储                | 可以像devicemapper一样直接操作底层设备，支持动态添加设备     | 不支持共享存储，当有多个容器读同一个文件时，需要生成多个复本 | 不适合在高密度容器的paas平台上使用 |
| ZFS          | 把所有设备集中到一个存储池中来进行管理   | 支持多个容器共享一个缓存块，适合内存大的环境                 | COW使用碎片化问题更加严重，文件在硬盘上的物理地址会变的不再连续，顺序读会变的性能比较差 | 适合paas和高密度的场景             |



- **AUFS VS OverlayFS**

AUFS和Overlay都是联合文件系统，但AUFS有多层，而Overlay只有两层，所以在做写时复制操作时，如果文件比较大且存在比较低的层，则AUSF可能会慢一些。而且Overlay并入了linux kernel mainline，AUFS没有。目前AUFS已基本被淘汰

- **OverlayFS VS Device mapper**

OverlayFS是文件级存储，Device mapper是块级存储，当文件特别大而修改的内容很小，Overlay不管修改的内容大小都会复制整个文件，对大文件进行修改显示要比小文件要消耗更多的时间，而块级无论是大文件还是小文件都只复制需要修改的块，并不是整个文件，在这种场景下，显然device mapper要快一些。因为块级的是直接访问逻辑盘，适合IO密集的场景。而对于程序内部复杂，大并发但少IO的场景，Overlay的性能相对要强一些。





# Docker存储卷使用

docker提供数据卷来实现数据共享与持久化，而数据卷的挂载有两种方式：

- 挂载主机目录（Bind mounts）
- 数据卷容器（Data Volumes）

数据卷是一个可供容器使用的特殊目录，它绕过文件系统，可以提供很多有用的特性：

- 数据卷可以在容器之间共享和重用
- 对数据卷的修改会立马生效
- 对数据卷的更新不会影响镜像
- 卷会一直存在，只到没有容器使用



**选择 -v 或 --mount 标志**

一般来说，`--mount`更明确和详细。最大的区别是`-v`语法将所有选项组合在一个字段中，而`--mount` 语法将它们分开。这是每个标志的语法比较。

如果需要指定卷驱动程序选项，则必须使用`--mount`.

```
从外部 CSV 解析器中转义值

如果您的卷驱动程序接受以逗号分隔的列表作为选项，则您必须从外部 CSV 解析器中转义该值。要转义 a volume-opt，请用双引号 ( ") 将其括起来，并用单引号 ( ) 将整个安装参数括起来'。

例如，local驱动程序接受挂载选项作为o参数中的逗号分隔列表。此示例显示了转义列表的正确方法。

$ docker service create \
    --mount 'type=volume,src=<VOLUME-NAME>,dst=<CONTAINER-PATH>,volume-driver=local,volume-opt=type=nfs,volume-opt=device=<nfs-server>:<nfs-path>,"volume-opt=o=addr=<nfs-address>,vers=4,soft,timeo=180,bg,tcp,rw"'
    --name myservice \
    <IMAGE>
```



## 创建和管理卷

与绑定挂载不同，您可以在任何容器范围之外创建和管理卷。

**创建卷**：

```
$ docker volume create my-vol
```

**列出卷**：

```
[root@localhost ~]# docker volume ls
DRIVER              VOLUME NAME
local               my-vol

```

**检查卷**：

```
[root@localhost ~]#  docker volume inspect my-vol
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
]

```

**删除卷**：

```
$ docker volume rm my-vol
```

## 启动一个带有卷的容器

如果您使用尚不存在的卷启动容器，Docker 会为您创建卷。以下示例将卷安装`myvol2`到 `/app/`容器中。

下面的`-v`和`--mount`示例产生相同的结果。除非在运行第一个容器和卷后删除`devtest`容器和卷，否则不能同时运行它们。`myvol2`

```bash
[root@localhost volumes]# docker run -d   --name devtest   --mount source=myvol2,target=/app   nginx:latest
unknown flag: --mount
See 'docker run --help'.
# 报错
docker run support for the --mount option was only introduced in Docker 17.06. You are using Docker 1.13.1. You have two choices:

Update to Docker 17.06 or later if you can;

Use the -v approach to bind mount the volume you require e.g. docker run -v $(pwd):/home

以上的意思是解决unknown flag: --mount 这个问题，有两个办法。

使用-v 或者使用17.06以上的版本。
```





## 挂载主机目录（Bind mounts）

**(1) 基本用法**

下面在容器启动时通过 **-v** 参数将 **docker host** 上的 **$HOME/htdocs** 目录 **mount**到 **httpd** 容器里。

```
docker run -d -v ~/htdocs:/usr/local/apache2/htdocs httpd
```

这里的 **/usr/local/apache2/htdocs** 就是 **Apache Server** 存放静态文件的地方。**mount** 后，容器中 **/usr/local/apache2/htdocs** 里原有数据会被隐藏起来，取而代之代之的是 **host $HOME/htdocs/** 中的数据。

**注意**：当我们 **mount** 一个目录时，如果更改宿主机目录里的内容，容器内对应的内容也会同步更新，不需要重启容器。



**(2) 指定数据的读写权限**

默认情况下 **mount** 的数据是可读可写的。我们可以添加 **ro** 参数设置成只读权限，此时：

- 在容器中无法对 **bind mount** 数据进行修改。 
- 只有 **host** 有权修改数据。

```
[root@localhost ~]# docker run -d -v ~/htdocs:/usr/local/apache2/htdocs:ro httpd
d266b4f5d70f8633051ef06433142f6292f72965be128509088198fb070a2607
# 发现在/usr/local/apache2/htdocs目录后多了ro权限位
```

(3) 单个文件映射

如果只需要向容器添加文件，不希望覆盖整个目录，可以使用 **bind mount** 单个文件。下面样例，我们将一个 **html** 文件添加到 **apache** 中，同时也保留了容器原有的数据后。

**注意**：如果我们 **mount** 的是单个文件的话，不同于 **mount** 整个目录，运行后如果宿主机的这个文件进行了修改，容器内对应的文件内容是不会同步变化的，必须重启容器才会更新。

```
docker run -d -v ~/htdocs/index.html:/usr/local/apache2/htdocs/new_index.html httpd
```



## 数据卷容器（Data Volumes）

**(1) 基本用法**

**docker managed volume** 和 **bind mount** 在使用上最大的区别是不需要指定 **mount** 源，指明 **mount point** 就好。
同样以 **httpd** 容器为例。我们通过 **-v** 告诉它需要一个 **data volume**，并将其 **mount** 到 **/usr/local/apache2/htdocs**。

```
docker run -d -v /usr/local/apache2/htdocs httpd
```

上面执行后，**docker** 就会自动在 **host** 的 **/var/lib/docker/volumes** 下生成一个目录，这个目录就是 **mount** 源。同时还会将容器里中 **/usr/local/apache2/htdocs** 数据复制到 **mount** 源中。

**(2) 查找 data volume 的具体位置**

要找到它我们先执行 **docker inspect** 查看容器配置信息：

```bash
[root@localhost ~]# docker run -d -v /usr/local/apache2/htdocs httpd
26c355977e048e520ce6ebc79469470494dd2a917b3bab1c11e3c596864a6631
[root@localhost ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND              CREATED             STATUS              PORTS               NAMES
26c355977e04        httpd               "httpd-foreground"   11 seconds ago      Up 10 seconds       80/tcp              practical_engelbart

# 找到mounts就能获取到宿主机映射的目录
[root@localhost ~]# docker inspect 26c355977e04
"Mounts": [
            {
                "Type": "volume",
                "Name": "31f3c299af00b97627ac9c0b53a1303fcfc29c2e6695d53ff2203def89d5f816",
                "Source": "/var/lib/docker/volumes/31f3c299af00b97627ac9c0b53a1303fcfc29c2e6695d53ff2203def89d5f816/_data",
                "Destination": "/usr/local/apache2/htdocs",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],


```

在 **Mounts** 这部分的信息中会显示容器当前所使用的所有 **data volume**，包括 **bind mount** 和 **docker managed volume**。

我们进这个文件夹可以看到，容器里中 **/usr/local/apache2/htdocs** 数据确实已经复制到这个 **mount** 源中。

```
[root@localhost volumes]# cd /var/lib/docker/volumes/31f3c299af00b97627ac9c0b53a1303fcfc29c2e6695d53ff2203def89d5f816/_data/
[root@localhost _data]# ll
total 4
-rw-r--r-- 1 504 ftp 45 Jun 12  2007 index.html

```

