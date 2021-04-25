---
layout: post
title: CentOS NFS 共享服务搭建
categories: CentOS
description: CentOS NFS 共享服务搭建
keywords: centos, nfs
---

## 0x00 写在前面
- 通过nfs共享磁盘给192.168.1.0/24网段内主机使用
- 已经格式化硬盘并mount到/data上

## 0x01 安装服务

```shell
yum install -y nfs-utils
```

## 0x02 修改nfs配置
```
mkdir /data
chmod 777 /data
echo -e "MOUNTD_PORT=4001\nSTATD_PORT=4002\nLOCKD_TCPPORT=4003\nLOCKD_UDPPORT=4003\nRQUOTAD_PORT=4004" >> /etc/sysconfig/nfs
echo -e "/data 192.168.1.0/24(rw)" >> /etc/exports
exportfs -r	#配置重新读取
exportfs -v
```

/etc/sysconfig/nfs 配置内容
```shell
......
MOUNTD_PORT=4001　　
STATD_PORT=4002
LOCKD_TCPPORT=4003
LOCKD_UDPPORT=4003
RQUOTAD_PORT=4004
```
/etc/exports配置内容
```shell
/data 192.168.1.0/24(rw)
```

exportfs -v 结果输出
```shell
[root@localhost ~]# exportfs -v
/data         	192.168.1.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
```

## 0x03 启动服务并设置开机自启动

```shell
systemctl enable rpcbind.service
systemctl start rpcbind.service
systemctl enable nfs.service 
systemctl start nfs.service 
systemctl enable nfs-server.service
systemctl start nfs-server.service
```

## 0x04 修改防火墙配置放行指定端口

```shell
firewall-cmd --zone=public --add-port=111/tcp --permanent 
firewall-cmd --zone=public --add-port=111/udp --permanent 
firewall-cmd --zone=public --add-port=2049/udp --permanent 
firewall-cmd --zone=public --add-port=2049/tcp --permanent 
firewall-cmd --zone=public --add-port=4001/tcp --permanent 
firewall-cmd --zone=public --add-port=4002/tcp --permanent 
firewall-cmd --zone=public --add-port=4003/tcp --permanent 
firewall-cmd --zone=public --add-port=4004/tcp --permanent 
firewall-cmd --zone=public --add-port=4004/udp --permanent 
firewall-cmd --zone=public --add-port=4003/udp --permanent 
firewall-cmd --zone=public --add-port=4002/udp --permanent 
firewall-cmd --zone=public --add-port=4001/udp --permanent 
firewall-cmd --reload
```

