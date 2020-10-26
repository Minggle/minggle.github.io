---
layout: post
title: CentOS SMB 无密码共享
categories: CentOS
description: CentOS SMB 无密码共享
keywords: centos, smb
---

## 0x00 需求
> CentOS系统共享文件夹给Windows使用，且无需密码。

## 0x01 查看是否安装

```shell
rpm -qi samba
```

## 0x02 安装samba服务及客户端

```shell
yum -y install samba samba-client
```

## 0x03 备份配置文件并写入新配置文件

```shell
mv /etc/samba/smb.conf /etc/samba/smb.conf.back
echo -e '[global]\n\tworkgroup = WORKGROUP\n\tsecurity = user\n\tmap to guest = Bad User\n\tpassdb backend = tdbsam\n\tprinting = cups\n\tprintcap name = cups\n\tload printers = yes\n\tcups options = raw\n[share]\n\tcomment = share all\n\tpath = /usr/local/src/zabbix_windows\n\tread only = yes\n\tguest ok = yes\n\tbrowseable = yes\n' >> /etc/samba/smb.conf
```

>配置文件说明
```shell
[global]
	workgroup = WORKGROUP	#设置工作组为Windows工作组
	security = user	#用户认证
	map to guest = Bad User	#映射为guest用户
	passdb backend = tdbsam
	printing = cups
	printcap name = cups
	load printers = yes
	cups options = raw
[share]
	comment = share all	#描述
	path = /usr/local/src/zabbix_windows	#共享文件路径
	read only = yes	#只读权限
	guest ok = yes	#guest无密码访问权限
	browseable = yes	#允许查看权限
```

## 0x04 启动smb服务并设置开机自启动

```shell
systemctl start smb
systemctl enable smb
```

## 0x05 防火墙放行samba服务

```shell
firewall-cmd --zone=public --add-service=samba --permanent
firewall-cmd --reload
```

## 0x06 测试配置文件及服务是否正常

```shell
testparm
smbclient -L 127.0.0.1
```

