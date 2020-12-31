---
layout: post
title: Zabbix Server 服务无法启动排错记录
categories: Zabbix
description: Zabbix Server 服务无法启动排错记录
keywords: zabbix, CacheSize
---

## Zabbix 正常运行过程中，突然服务停止，无法启动，重启不生效。

```shell
[root@zabbix ~]# service zabbix-server restart
Redirecting to /bin/systemctl restart zabbix-server.service
[root@zabbix ~]# service zabbix-server status
Redirecting to /bin/systemctl status zabbix-server.service
● zabbix-server.service - Zabbix Server
   Loaded: loaded (/usr/lib/systemd/system/zabbix-server.service; enabled; vendor preset: disabled)
   Active: activating (auto-restart) (Result: exit-code) since Tue 2019-12-03 09:56:32 CST; 6s ago
  Process: 870 ExecStop=/bin/kill -SIGTERM $MAINPID (code=exited, status=1/FAILURE)
  Process: 859 ExecStart=/usr/sbin/zabbix_server -c $CONFFILE (code=exited, status=0/SUCCESS)
 Main PID: 862 (code=exited, status=1/FAILURE)

Dec 03 09:56:32 zabbix systemd[1]: zabbix-server.service: control process exited, code=exited status=1
Dec 03 09:56:32 zabbix systemd[1]: Unit zabbix-server.service entered failed state.
Dec 03 09:56:32 zabbix systemd[1]: zabbix-server.service failed.
```

## 查看日志，发现CacheSize 配置报错

```shell
[root@zabbix ~]# tailf /var/log/zabbix/zabbix_server.log
***
  1090:20191203:095931.318 [file:dbconfig.c,line:94] __zbx_mem_realloc(): out of memory (requested 2592 bytes)
  1090:20191203:095931.318 [file:dbconfig.c,line:94] __zbx_mem_realloc(): please increase CacheSize configuration parameter
***
```

## 修改CacheSize配置

```shell
[root@zabbix ~]# vim /etc/zabbix/zabbix_server.conf
***
### Option: CacheSize
#       Size of configuration cache, in bytes.
#       Shared memory size for storing host, item and trigger data.
#
# Mandatory: no
# Range: 128K-8G
# Default:
CacheSize=2G
***
```

## 重启Zabbix_server服务

```shell
[root@zabbix ~]# service zabbix-server restart
Redirecting to /bin/systemctl restart zabbix-server.service
[root@zabbix ~]# service zabbix-server status
Redirecting to /bin/systemctl status zabbix-server.service
● zabbix-server.service - Zabbix Server
   Loaded: loaded (/usr/lib/systemd/system/zabbix-server.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2019-12-03 10:03:12 CST; 6s ago
  Process: 8044 ExecStop=/bin/kill -SIGTERM $MAINPID (code=exited, status=0/SUCCESS)
  Process: 8050 ExecStart=/usr/sbin/zabbix_server -c $CONFFILE (code=exited, status=0/SUCCESS)
 Main PID: 8052 (zabbix_server)

```
## 启动成功。

