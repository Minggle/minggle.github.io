---
layout: post
title: Nessus 8.12.0 安装破解
categories: Security
description: Nessus 8.12.0 安装破解
keywords: nessus
---

# Nessus 8.12.0 安装破解

## 1. 下载Nessus

> 根据基础环境进行选择下载，下载地址](https://www.tenable.com/downloads/nessus)。

我的环境是CentOS 7.6 x64，所以下载的是Nessus-8.12.0-es7.x86_64.rpm。

![image-20201029142706482](https://raw.githubusercontent.com/Minggle/image/main/image/image-20201029142706482.png)

## 2. 安装

上传到服务器中，我的上传路径是`/usr/local/src/`

```shell
[root@nessus src]# cd /usr/local/src/
[root@nessus src]# ls -alh | grep Ness
-rw-r--r--.  1 root root 38M Oct 29 14:29 Nessus-8.12.0-es7.x86_64.rpm
```

执行安装程序

```shell
[root@nessus src]# rpm -ivh Nessus-8.12.0-es7.x86_64.rpm
warning: Nessus-8.12.0-es7.x86_64.rpm: Header V4 RSA/SHA1 Signature, key ID 1c0c4a5d: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:Nessus-8.12.0-es7                ################################# [100%]
Unpacking Nessus Core Components...
 - You can start Nessus by typing /bin/systemctl start nessusd.service
 - Then go to https://nessus:8834/ to configure your scanner
```

放开防火墙8834端口

```shell
[root@nessus src]# firewall-cmd --zone=public --add-port=8834/tcp --permanent
success
[root@nessus src]# firewall-cmd --reload
success
[root@nessus src]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources:
  services: ssh dhcpv6-client
  ports: 8834/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

启动nessus进程

```shell
[root@nessus src]# systemctl start nessusd.service
[root@nessus src]# ps -ef | grep nessus | grep -v grep
root      86658      1  0 14:34 ?        00:00:00 /opt/nessus/sbin/nessus-service -q
root      86659  86658 70 14:34 ?        00:00:40 nessusd -q
[root@nessus src]# netstat -pantu | grep nessus
tcp        0      0 0.0.0.0:8834            0.0.0.0:*               LISTEN      86659/nessusd
tcp6       0      0 :::8834                 :::*                    LISTEN      86659/nessusd
```



## 3. 初始化设置

访问服务器8834端口，访问地址：

https://[nessus ip]:8834

![image-20201029143847614](https://raw.githubusercontent.com/Minggle/image/main/image/image-20201029143847614.png)

![image-20201029143942717](https://raw.githubusercontent.com/Minggle/image/main/image/image-20201029143942717.png)

![image-20201029144003583](https://raw.githubusercontent.com/Minggle/image/main/image/image-20201029144003583.png)

![image-20201029144027407](https://raw.githubusercontent.com/Minggle/image/main/image/image-20201029144027407.png)

![image-20201029144043942](https://raw.githubusercontent.com/Minggle/image/main/image/image-20201029144043942.png)

![image-20201029144142114](https://raw.githubusercontent.com/Minggle/image/main/image/image-20201029144142114.png)

## 4. 获取离线插件包

获取离线code

```shell
[root@nessus src]# /opt/nessus/sbin/nessuscli fetch --challenge

Challenge code: b9c940cb14059d0xxxxxxxxxxxxxxx469

You can copy the challenge code above and paste it alongside your
Activation Code at:
https://plugins.nessus.org/v2/offline.php
```

获取active code

[https://zh-cn.tenable.com/products/nessus/nessus-essentials](https://zh-cn.tenable.com/products/nessus/nessus-essentials)

姓名可以乱填，邮箱要是真的，active code会发送到邮箱。

![image-20201029145540095](https://raw.githubusercontent.com/Minggle/image/main/image/image-20201029145540095.png)

收到一封邮件，获取active code

![image-20201029145641284](https://raw.githubusercontent.com/Minggle/image/main/image/image-20201029145641284.png)

访问[https://plugins.nessus.org/v2/offline.php](https://plugins.nessus.org/v2/offline.php)获取离线包

![image-20201029145821206](https://raw.githubusercontent.com/Minggle/image/main/image/image-20201029145821206.png)

获取授权和离线安装包下载路径

![image-20201029145915945](https://raw.githubusercontent.com/Minggle/image/main/image/image-20201029145915945.png)





## 5. 导入授权

```shell
[root@nessus src]# /opt/nessus/sbin/nessuscli fetch --register-offline /usr/local/src/nessus.license
Your Activation Code has been registered properly - thank you.

```

## 6. 离线更新插件

```SHELL
[root@nessus src]# /opt/nessus/sbin/nessuscli update /usr/local/src/all-2.0.tar.gz

[info] Copying templates version 202010191623 to /opt/nessus/var/nessus/templates/tmp
[info] Finished copying templates.
[info] Moved new templates with version 202010191623 from plugins dir.
 * Update successful.  The changes will be automatically processed by Nessus. 
```

上面的`version`版本要记住，下面要用。

## 7. 破解

```shell
mv /opt/nessus/lib/nessus/plugins/plugin_feed_info.inc /tmp		##移除插件目录下的plugin_feed_info.inc
touch /opt/nessus/var/nessus/plugin_feed_info.inc		##创建破解文件（如文件不存在）
mkdir -p /opt/plugins_backup_2020 && cp -r  /opt/nessus/lib/nessus/plugins /opt/plugins_backup_2020/		## 备份插件
echo -e "PLUGIN_SET = \"202010290423\";\nPLUGIN_FEED = \"ProfessionalFeed (Direct)\";\nPLUGIN_FEED_TRANSPORT = \"Tenable Network Security Lightning\";" >/opt/nessus/var/nessus/plugin_feed_info.inc		##PLUGIN_SET 使用获取到的`version`替换
systemctl restart nessusd.service		## 重启服务
```

`plugin_feed_info.inc`内容如下：

```shell
PLUGIN_SET = "202010191623";
PLUGIN_FEED = "ProfessionalFeed (Direct)";
PLUGIN_FEED_TRANSPORT = "Tenable Network Security Lightning";
```



## 8. 登录使用

https://[nessus ip]:8834

![image-20201102114429102](https://raw.githubusercontent.com/Minggle/image/main/image/image-20201102114429102.png)


