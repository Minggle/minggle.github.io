---
layout: post
title: Ubuntu 18.04 Server 修改网络配置
categories: Ubuntu
description: Ubuntu 18.04 Server 修改网络配置
keywords: ubuntu, 网络
---

## 前言
有业务需求需要修改Ubuntu 18.04 server的网络地址，使用原有方式无法生效。

## ~~原有配置方式~~
> 自Ubuntu 17.10 开始，已经不支持使用此方式进行了。


- 修改配置信息


```shell
vim /etc/network/interfaces
……
auto eth0 # eth0网卡随系统自启动
iface eth0 inet static # eth0为静态地址
address 192.168.2.147 # 配置eth0地址
netmask 255.255.255.0 # 配置eth0子网掩码
gateway 192.168.2.1 # 配置eth0默认网关
dns-nameservers 223.5.5.5 # 配置域名解析地址
……
```
- 重启网卡服务

```shell
/etc/init.d/networking restart
```

- 查看路由表

```shell
route -n
```

## 现配置方式

> 自Ubuntu 17.10 开始，使用netplan进行网络配置

- 修改配置文件

```shell
vim /etc/netplan/50-cloud-init.yaml	# 配置文件分层格式
……
network:
    ethernets:
        ens33:
            addresses: [ "192.168.1.2/24" ]	# 配置IP地址及子网掩码
            dhcp4: false	# 静态地址
            optional: true	
            gateway4: 192.168.1.1	# 默认网关
            nameservers:	# DNS服务
                    addresses: [223.5.5.5]	# DNS服务地址
    version: 2
……
```
- 应用netplan

```shell
sudo netplan apply
```

- 查看网卡及路由信息

```shell
ifconfig	# 查看地址信息
route -n	# 查看路由信息

```

