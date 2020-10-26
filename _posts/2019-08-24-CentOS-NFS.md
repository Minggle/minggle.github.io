---
layout: post
title: Centos NFS 挂载
categories: CentOS
description: Centos NFS 挂载
keywords: centos, NFS， 挂载
---

# 0x01 安装nfs-utils和rpcbind

```
yum install nfs-utils rpcbind -y
```
# 0x02 设置开机启动服务

```
chkconfig nfs on
chkconfig rpcbind on
```
# 0x03 启动服务

```
service rpcbind start
service nfs start
```
# 0x04 创建挂载点

```
mkdir -p /mnt/share
```
# 0x05 挂载目录

```
mount -t nfs server_ip:/share /mnt/share    /mnt/share
```
# 0x06 查看挂载的目录

```
df -h
```
# 0x07 卸载挂载的目录

```
umount /mnt/share
```
# 0x08 编辑/etc/fstab，开机自动挂载

```
vim /etc/fstab
# 在结尾添加如下一行
server_ip:/share /mnt/share nfs rw,tcp,intr 0 1
```