



# CentOS7磁盘无法删除分区



```
[root@host-10-18-5-123 ~]# lsblk -af
NAME   FSTYPE LABEL UUID                                 MOUNTPOINT
sr0                                                      
vda                                                      
├─vda1 ext4         01c7237a-ba0d-4f13-8056-b9f1db41ca84 /boot
├─vda2 ext4         18fe8ddb-bcfd-4b18-a70d-eb0ee97f247b /
└─vda3 swap         ac04a9dd-abb9-4b7e-974c-5d5c007676d7 
vdb                                                      
└─vdb1                                                   
[root@host-10-18-5-123 ~]# fdisk /dev/vdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x77ded59c.

Command (m for help): p

Disk /dev/vdb: 536.9 GB, 536870912000 bytes, 1048576000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x77ded59c

   Device Boot      Start         End      Blocks   Id  System

Command (m for help): d
No partition is defined yet!

Command (m for help): 
```

发现磁盘/dev/vdb有分区但是fdisk无法删除。







```
[root@host-10-18-5-123 ~]# cat /proc/partitions 
major minor  #blocks  name

 253        0  188743680 vda
 253        1   20971520 vda1
 253        2  104857600 vda2
 253        3   62913536 vda3
 253       16  524288000 vdb
 253       17  524286976 vdb1
  11        0    1048575 sr0
```

