---
layout: post
title: CentOS 7 Firewalld 笔记
categories: CentOS
description:  CentOS 7 Firewalld 笔记
keywords: CentOS , Firewalld 
---

# CentOS 7 Firewalld 笔记

## 1. 系统控制

### 1.1 启动

```shell
systemctl start firewalld
```

## 1.2 停止

```shell
systemctl stop firewalld
```

### 1.3 查看状态

```shell
systemctl status firewalld
```

### 1.4 设置开机自启动

```shell
systemctl enable firewalld
```

### 1.5 取消开机自启

```shell
systemctl disable firewalld
```



## 2. 防火墙基础配置

```shell
--permanent    # 永久生效可重起
```

### 2.1 放行特定端口

```shell
firewall-cmd --zone=public --add-port=6000/tcp --permanent
```
### 2.2 取消放行特定端口
```shell
firewall-cmd --zone=public --remove-port=6000/tcp --permanent
```

### 2.3 放行特定服务

```shell
firewall-cmd --zone=public --add-service=https --permanent
```
### 2.4 取消放行特定服务

```shell
firewall-cmd --zone=public --remove-service=https --permanent
```

### 2.5 重新载入配置
```shell
firewall-cmd --reload
```
### 2.5 查看放行的端口
```shell
firewall-cmd --list-port
```
### 2.5 查看全部配置
```shell
firewall-cmd --list-all
```



## 3. 防火墙进阶配置

### 3.1 只允许特定ip访问本地特定端口
```shell
firewall-cmd --add-rich-rule="rule family="ipv4" source address="192.168.1.100" port protocol="tcp" port="7001" accept" --permanent
```

### 3.2 不允许特定ip访问本地特定端口

```shell
firewall-cmd --add-rich-rule="rule family="ipv4" source address="192.168.1.100" port protocol="tcp" port="7001" drop" --permanent
```

### 3.3 删除rich-rule规则

```shell
firewall-cmd --remove-rich-rule="rule family="ipv4" source address="192.168.1.100" port protocol="tcp" port="7001" accept" --permanent
```

### 3.4 端口转发

```shell
# firewall-cmd --zone=[区域名]  --add-forward-port=port=[被转发端口]:proto=tcp:toport=[转发后端口]   
# eg.访问80端口被转发到7001 
firewall-cmd --zone=public --add-forward-port=port=80:proto=tcp:toport=7001 --permanent 
```

### 3.5 四个zone

-   `public`:仅允许访问本机的ssh dhcp ping服务
-   `trusted`:允许任何访问
-   `block`:阻塞任何来访请求(明确拒绝,有回应客户端)
-   `drop`:丢弃任何来访的数据包(没有回应,节省服务端资源)

```shell
# 查看当前区域
firewall-cmd --get-default-zone

# 设置默认区域
firewall-cmd --set-default-zone='trusted'

# 查看区域配置
firewall-cmd --zone='trusted' --list-all

# 查看全部区域配置
firewall-cmd  --list-all-zone

# 添加源
firewall-cmd --zone='trusted' --add-source='192.168.0.100'
```

