---
layout: post
title: Zabbix 密码忘记&重置
categories: Zabbix
description: Zabbix 密码忘记&重置
keywords: zabbix, password
---

# 0x00 Zabbix密码忘了
>自己搭建的zabbix测试服务器，由于近期比较忙，一直没有登陆操作。
>近期时间稍空，准备继续折腾，发现登陆用户名及密码忘记了，所以进行了密码重置，本文用于记录使用。
>文中所涉及到的密码、md5均为演示使用。

![](http://cdn.mingsec.com/zabbix_1.jpg)


# 0x01 查询数据库中密码

- 登陆服务器
- 登陆数据库

```shell
[root@zabbix ~]# mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 187
Server version: 5.5.60-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

- 查询数据库

```shell
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| zabbix             |
+--------------------+
4 rows in set (0.01 sec)

```

- 使用数据库

```shell
MariaDB [(none)]> use zabbix;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

```

- 查询用户名密码信息

```shell
MariaDB [zabbix]> select alias,name,passwd from users;
+-------+--------+----------------------------------+
| alias | name   | passwd                           |
+-------+--------+----------------------------------+
| Admin | Zabbix | e10adc3949ba59abbe56e057f20f883e |
| guest |        | d41d8cd98f00b204e9800998ecf8427e |
+-------+--------+----------------------------------+

```

# 0x02 破解密码

> 由于数据库密码存储为md5加密，如果密码不复杂，可以直接在线破解
> [https://www.somd5.com/](https://www.somd5.com/)

![](http://cdn.mingsec.com/zabbix_2.jpg)


> 如果密码强度过高，则无法暴力破解，此时需要重置密码

```shell
MariaDB [zabbix]> update users set passwd='5fce1b3e34b520afeffb37ce08c7cd66' where alias='Admin';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

MariaDB [zabbix]> select alias,name,passwd from users;
+-------+--------+----------------------------------+
| alias | name   | passwd                           |
+-------+--------+----------------------------------+
| Admin | Zabbix | 5fce1b3e34b520afeffb37ce08c7cd66 |
| guest |        | d41d8cd98f00b204e9800998ecf8427e |
+-------+--------+----------------------------------+
2 rows in set (0.00 sec)

```

# 0x03 重置完成

>username:Admin
>password:zabbix

