---
layout: post
title: wordpress 数据库自动关闭修复
categories: Wordpress
description: wordpress 数据库自动关闭修复
keywords: wordpress, 数据库, 关闭
---

## 查看数据库报错日志
less /var/log/mariadb/mariadb.log
```shell
200213 04:47:09 mysqld_safe Number of processes running now: 0
200213 04:47:09 mysqld_safe mysqld restarted
200213  4:47:10 [Note] /usr/libexec/mysqld (mysqld 5.5.60-MariaDB) starting as process 4402 ...
200213  4:47:10 [ERROR] mysqld: Out of memory (Needed 128917504 bytes)
200213  4:47:10 [ERROR] mysqld: Out of memory (Needed 96681984 bytes)
200213  4:47:10 InnoDB: The InnoDB memory heap is disabled
200213  4:47:10 InnoDB: Mutexes and rw_locks use GCC atomic builtins
200213  4:47:10 InnoDB: Compressed tables use zlib 1.2.7
200213  4:47:10 InnoDB: Using Linux native AIO
200213  4:47:10 InnoDB: Initializing buffer pool, size = 128.0M
InnoDB: mmap(137756672 bytes) failed; errno 12
200213  4:47:10 InnoDB: Completed initialization of buffer pool
200213  4:47:10 InnoDB: Fatal error: cannot allocate memory for the buffer pool
200213  4:47:10 [ERROR] Plugin 'InnoDB' init function returned error.
200213  4:47:10 [ERROR] Plugin 'InnoDB' registration as a STORAGE ENGINE failed.
200213  4:47:10 [Note] Plugin 'FEEDBACK' is disabled.
200213  4:47:10 [ERROR] Unknown/unsupported storage engine: InnoDB
200213  4:47:10 [ERROR] Aborting
```
## 创建SWAP分区
```shell
dd if=/dev/zero of=/var/swap bs=1024 count=4194304
mkswap /var/swap
swapon /var/swap

vim /etc/fstab
/var/swap	swap	swap	defaults	0 0

reboot now
```
## 编写数据库自动重启进程

vim /etc/systemd/system/mariadb.service
```shell
.include /lib/systemd/system/mariadb.service

[Service]
Restart=always
RestartSec=3
```


## 重启服务
```shell
systemctl daemon-reload
systemctl restart mariadb
systemctl restart php-fpm.service
```

