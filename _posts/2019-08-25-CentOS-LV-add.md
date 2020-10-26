---
layout: post
title: CentOS LV 磁盘扩容
categories: CentOS
description: CentOS LV 磁盘扩容
keywords: centos, lv, 扩容
---

# 0x01 查看新添加硬盘信息

```
[root@localhost ~]# fdisk -l

Disk /dev/sda: 17.2 GB, 17179869184 bytes
64 heads, 32 sectors/track, 16384 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0000c6f7

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *           2        1025     1048576   83  Linux
Partition 1 does not end on cylinder boundary.
/dev/sda2            1026        9217     8388608   82  Linux swap / Solaris
Partition 2 does not end on cylinder boundary.
/dev/sda3            9218       16384     7339008   8e  Linux LVM
Partition 3 does not end on cylinder boundary.

Disk /dev/sdb: 214.7 GB, 214748364800 bytes
255 heads, 63 sectors/track, 26108 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xd33ed81a

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1               1       26108   209712478+  8e  Linux LVM

Disk /dev/mapper/VolGroup-LogVol00: 7511 MB, 7511998464 bytes
255 heads, 63 sectors/track, 913 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000


Disk /dev/mapper/vg_sso-lv_sso: 109.1 GB, 109051904000 bytes
255 heads, 63 sectors/track, 13258 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000


Disk /dev/mapper/vg_sso-lv_postgres: 105.7 GB, 105692266496 bytes
255 heads, 63 sectors/track, 12849 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000


Disk /dev/sdc: 10.7 GB, 10737418240 bytes
64 heads, 32 sectors/track, 10240 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

```

# 0x02 格式化硬盘

```
[root@localhost ~]# fdisk /dev/sdc
Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
Building a new DOS disklabel with disk identifier 0x0e79fdf0.
Changes will remain in memory only, until you decide to write them.
After that, of course, the previous content won't be recoverable.

Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
         switch off the mode (command 'c') and change display units to
         sectors (command 'u').

Command (m for help): p

Disk /dev/sdc: 10.7 GB, 10737418240 bytes
64 heads, 32 sectors/track, 10240 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0e79fdf0

   Device Boot      Start         End      Blocks   Id  System

Command (m for help): n
Command action
   e   extended
   p   primary partition (1-4)
e
Partition number (1-4): 1
First cylinder (1-10240, default 1): 
Using default value 1
Last cylinder, +cylinders or +size{K,M,G} (1-10240, default 10240): 
Using default value 10240

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.

[root@localhost ~]# fdisk /dev/sdc

WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
         switch off the mode (command 'c') and change display units to
         sectors (command 'u').

Command (m for help): p

Disk /dev/sdc: 10.7 GB, 10737418240 bytes
64 heads, 32 sectors/track, 10240 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x1fc4971f

   Device Boot      Start         End      Blocks   Id  System
/dev/sdc1               1       10240    10485744    5  Extended

Command (m for help): n
Command action
   l   logical (5 or over)
   p   primary partition (1-4)
l  
First cylinder (1-10240, default 1): 
Using default value 1
Last cylinder, +cylinders or +size{K,M,G} (1-10240, default 10240): 
Using default value 10240

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
[root@localhost ~]# pvcreate /dev/sdc5
  Physical volume "/dev/sdc5" successfully created
[root@localhost ~]# pvdisplay /dev/sdc5
  "/dev/sdc5" is a new physical volume of "10.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdc5
  VG Name               
  PV Size               10.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               v30kl9-9W7b-rjIe-UN5D-bm8l-mR68-reNPA1
```

# 0x03 扩展磁盘到VG

```
[root@localhost ~]# vgextend VolGroup /dev/sdc5
  Volume group "VolGroup" successfully extended
[root@localhost ~]# vgdisplay VolGroup
  --- Volume group ---
  VG Name               VolGroup
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               16.99 GiB
  PE Size               4.00 MiB
  Total PE              4350
  Alloc PE / Size       1791 / 7.00 GiB
  Free  PE / Size       2559 / 10.00 GiB
  VG UUID               ipWL1f-t2OB-jgRp-wYp3-1I1c-Wz0T-Z3s0dD
   
[root@localhost ~]# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-LogVol00
                      6.8G  6.8G     0 100% /
tmpfs                 7.8G   16K  7.8G   1% /dev/shm
/dev/sda1             976M  141M  785M  16% /boot
/dev/mapper/vg_sso-lv_sso
                      100G   42G   54G  44% /home/sso
/dev/mapper/vg_sso-lv_postgres
                       97G   19G   74G  20% /home/postgres
```

# 0x04 添加磁盘空间

```
[root@localhost ~]# pvs
  PV         VG       Fmt  Attr PSize   PFree 
  /dev/sda3  VolGroup lvm2 a--u   7.00g     0 
  /dev/sdb1  vg_sso   lvm2 a--u 200.00g     0 
  /dev/sdc5  VolGroup lvm2 a--u  10.00g 10.00g

[root@localhost ~]# lvextend -L +10G /dev/VolGroup/LogVol00 
Insufficient free space: 2560 extents needed, but only 2559 available
[root@localhost ~]# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-LogVol00
                      6.8G  6.8G     0 100% /
tmpfs                 7.8G   16K  7.8G   1% /dev/shm
/dev/sda1             976M  141M  785M  16% /boot
/dev/mapper/vg_sso-lv_sso
                      100G   42G   54G  44% /home/sso
/dev/mapper/vg_sso-lv_postgres
                       97G   19G   74G  20% /home/postgres
[root@localhost ~]# lvextend -L +9G /dev/VolGroup/LogVol00 
  Size of logical volume VolGroup/LogVol00 changed from 7.00 GiB (1791 extents) to 16.00 GiB (4095 extents).
  Logical volume LogVol00 successfully resized.
[root@localhost ~]# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-LogVol00
                      6.8G  6.8G     0 100% /
tmpfs                 7.8G   16K  7.8G   1% /dev/shm
/dev/sda1             976M  141M  785M  16% /boot
/dev/mapper/vg_sso-lv_sso
                      100G   42G   54G  44% /home/sso
/dev/mapper/vg_sso-lv_postgres
                       97G   19G   74G  20% /home/postgres
```

# 0x05 刷新到文件系统

```
[root@localhost ~]# resize2fs -p /dev/VolGroup/LogVol00
resize2fs 1.41.12 (17-May-2010)
Filesystem at /dev/VolGroup/LogVol00 is mounted on /; on-line resizing required
old desc_blocks = 1, new_desc_blocks = 1
Performing an on-line resize of /dev/VolGroup/LogVol00 to 4193280 (4k) blocks.
The filesystem on /dev/VolGroup/LogVol00 is now 4193280 blocks long.

[root@localhost ~]# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-LogVol00
                       16G  6.8G  8.1G  46% /
tmpfs                 7.8G   16K  7.8G   1% /dev/shm
/dev/sda1             976M  141M  785M  16% /boot
/dev/mapper/vg_sso-lv_sso
                      100G   42G   54G  44% /home/sso
/dev/mapper/vg_sso-lv_postgres
                       97G   19G   74G  20% /home/postgres
```

# 0x06 添加剩余空间

```
[root@localhost ~]# pvs
  PV         VG       Fmt  Attr PSize   PFree   
  /dev/sda3  VolGroup lvm2 a--u   7.00g       0 
  /dev/sdb1  vg_sso   lvm2 a--u 200.00g       0 
  /dev/sdc5  VolGroup lvm2 a--u  10.00g 1020.00m
[root@localhost ~]# lvextend -L +1020M /dev/VolGroup/LogVol00 
  Size of logical volume VolGroup/LogVol00 changed from 16.00 GiB (4095 extents) to 16.99 GiB (4350 extents).
  Logical volume LogVol00 successfully resized.
[root@localhost ~]# resize2fs -p /dev/VolGroup/LogVol00
resize2fs 1.41.12 (17-May-2010)
Filesystem at /dev/VolGroup/LogVol00 is mounted on /; on-line resizing required
old desc_blocks = 1, new_desc_blocks = 2
Performing an on-line resize of /dev/VolGroup/LogVol00 to 4454400 (4k) blocks.
The filesystem on /dev/VolGroup/LogVol00 is now 4454400 blocks long.

[root@localhost ~]# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-LogVol00
                       17G  6.8G  9.1G  43% /
tmpfs                 7.8G   16K  7.8G   1% /dev/shm
/dev/sda1             976M  141M  785M  16% /boot
/dev/mapper/vg_sso-lv_sso
                      100G   42G   54G  44% /home/sso
/dev/mapper/vg_sso-lv_postgres
                       97G   19G   74G  20% /home/postgres
[root@localhost ~]# pvs
  PV         VG       Fmt  Attr PSize   PFree
  /dev/sda3  VolGroup lvm2 a--u   7.00g    0 
  /dev/sdb1  vg_sso   lvm2 a--u 200.00g    0 
  /dev/sdc5  VolGroup lvm2 a--u  10.00g    0 


```
# 报错时

```
[root@localhost ~]# resize2fs -p /dev/centos/root 
resize2fs 1.42.9 (28-Dec-2013)
resize2fs: Bad magic number in super-block while trying to open /dev/centos/root
Couldn't find valid filesystem superblock.
[root@localhost ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root  7.0G  6.1G  930M  88% /
devtmpfs                 3.9G     0  3.9G   0% /dev
tmpfs                    3.9G  8.0K  3.9G   1% /dev/shm
tmpfs                    3.9G  8.9M  3.9G   1% /run
tmpfs                    3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/sda1               1014M  311M  704M  31% /boot
tmpfs                    783M     0  783M   0% /run/user/0
[root@localhost ~]# xfs_growfs /dev/centos/root 
meta-data=/dev/mapper/centos-root isize=512    agcount=4, agsize=458496 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=1833984, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 1833984 to 28047360
[root@localhost ~]# 
[root@localhost ~]# 
[root@localhost ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root  107G  6.1G  101G   6% /
devtmpfs                 3.9G     0  3.9G   0% /dev
tmpfs                    3.9G  8.0K  3.9G   1% /dev/shm
tmpfs                    3.9G  8.9M  3.9G   1% /run
tmpfs                    3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/sda1               1014M  311M  704M  31% /boot
tmpfs                    783M     0  783M   0% /run/user/0
```