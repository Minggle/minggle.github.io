---
layout: post
title: CentOS 添加VG&LV
categories: CentOS
description: CentOS 添加VG&LV
keywords: centos, vg, lv, 扩容
---

# 0x01 查看硬盘信息
```shell
[root@localhost ~]# fdisk -l

Disk /dev/sdb: 322.1 GB, 322122547200 bytes, 629145600 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sda: 68.7 GB, 68719476736 bytes, 134217728 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000c29f7

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200   134217727    66059264   8e  Linux LVM

Disk /dev/mapper/centos-root: 59.1 GB, 59051606016 bytes, 115335168 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/centos-swap: 8589 MB, 8589934592 bytes, 16777216 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

# 0x02 格式化硬盘
```shell
[root@localhost ~]# fdisk /dev/sdb 
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1): 
First sector (2048-629145599, default 2048): 
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-629145599, default 629145599): 
Using default value 629145599
Partition 1 of type Linux and of size 300 GiB is set

Command (m for help): p

Disk /dev/sdb: 322.1 GB, 322122547200 bytes, 629145600 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xe433283a

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048   629145599   314571776   83  Linux

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
[root@localhost ~]# fdisk /dev/sdb 
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): t
Selected partition 1
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
[root@localhost ~]# fdisk /dev/sdb 
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): p

Disk /dev/sdb: 322.1 GB, 322122547200 bytes, 629145600 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xe433283a

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048   629145599   314571776   8e  Linux LVM

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```
# 0x03 创建并查看PV信息
```shell
[root@localhost ~]# pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created
[root@localhost ~]# pvdisplay 
  --- Physical volume ---
  PV Name               /dev/sda2
  VG Name               centos
  PV Size               <63.00 GiB / not usable 3.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              16127
  Free PE               0
  Allocated PE          16127
  PV UUID               MrM21N-bBbu-yqPu-Yse4-kJZx-231y-O8tTwo
   
  "/dev/sdb1" is a new physical volume of "<300.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb1
  VG Name               
  PV Size               <300.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               K3Hva7-0fda-pg89-4oIm-6pz4-1Wg0-karheN
```

# 0x04 创建VG

``` shell
[root@localhost ~]# vgcreate VG_data /dev/sdb1
  Volume group "VG_data" successfully created
```
# 0x05 创建LV
```shell
[root@localhost ~]# lvcreate -L 299G -n data VG_data
  Logical volume "data" created.
[root@localhost ~]# vgdisplay VG_data 
  --- Volume group ---
  VG Name               VG_data
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <300.00 GiB
  PE Size               4.00 MiB
  Total PE              76799
  Alloc PE / Size       76544 / 299.00 GiB
  Free  PE / Size       255 / 1020.00 MiB
  VG UUID               jVBQj4-F2Vb-JNXC-pwVv-TMmw-9w1W-bzDevo
   
```

# 0x06 格式化lv

- xfs格式，CentOS7以上版本使用

```shell
[root@localhost ~]# mkfs.xfs /dev/mapper/VG_data-data 
meta-data=/dev/mapper/VG_data-data isize=512    agcount=4, agsize=19660544 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=78642176, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=38399, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

```

- ext4格式，CentOS6使用

```shell
[root@localhost ~]# mkfs.ext4 /dev/mapper/DATA-App
mke2fs 1.41.12 (17-May-2010)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
22872064 inodes, 91488256 blocks
4574412 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=4294967296
2792 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968

Writing inode tables: done
Creating journal (32768 blocks): 
done
Writing superblocks and filesystem accounting information: 
done

This filesystem will be automatically checked every 32 mounts or
180 days, whichever comes first.  Use tune2fs -c or -i to override.
```

# 0x07 挂载lv到目录

```shell
[root@localhost ~]# mount /dev/mapper/VG_data-data /opt

```

# 0x08 添加开机自动挂载

```shell
[root@localhost ~]# vim /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Tue Jun  4 07:20:42 2019
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=9ac61ea5-20ad-4b96-87c5-9ba097a7b91b /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
/dev/mapper/VG_data-data	/opt	xfs	defaults	0 0
```

# 0x09 添加完成

```shell
[root@localhost ~]# df -h
Filesystem                Size  Used Avail Use% Mounted on
/dev/mapper/centos-root    55G  4.6G   51G   9% /
devtmpfs                  3.9G     0  3.9G   0% /dev
tmpfs                     3.9G     0  3.9G   0% /dev/shm
tmpfs                     3.9G  9.6M  3.9G   1% /run
tmpfs                     3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/mapper/VG_data-data  300G   44M  300G   1% /opt
/dev/sda1                1014M  271M  744M  27% /boot
tmpfs                     783M  8.0K  783M   1% /run/user/42
tmpfs                     783M     0  783M   0% /run/user/0
```

