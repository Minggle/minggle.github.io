---
layout: wiki
title: Mysql 学习笔记
categories: Mysql
description: Mysql 学习笔记
keywords: mysql
---

> 数据库纯小白，学习mysql记录下

## 1. 数据库安装
> 由于mysql被oracle收购，且在yum源中没法直接安装，现在使用mariadb比较方便
```shell
yum -y install mariadb mariadb-server
systemctl enable mariabd
systemctl start mariadb.service
```
> 安全初始化
```shell
mysql_secure_installation
```
> 安全初始化过程
```shell
[root@mysql ~]# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n]	#Y

New password:	#password
Re-enter new password:	#password
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n]
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] n
 ... skipping.

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n]	#Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n]	#Y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```
> 放通防火墙端口
```shell
firewall-cmd --zone=public --add-service=mysql --permanent
firewall-cmd --reload
```


## 2. 配置数据库远程登录权限

```shell
[root@mysql ~]# mysql -uroot -p
Enter password:	#password
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 11
Server version: 5.5.65-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> grant all on *.* to 'root'@'%' identified by 'password';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit
Bye
```



## 3. 数据库登录

```shell
mysql -u[username] -h[host] -p[password]
eg.
mysql -uroot -p
mysql -uroot -h 127.0.0.1 -p
mysql -uroot -hlocalhost -p
mysql -uroot -hdemo.aliyun.cn -p
```

## 4. 创建数据库

```sql
create database [dbname];
```

## 5. 创建数据表

```sql
CREATE TABLE IF NOT EXISTS `csdn`(
   `id` INT UNSIGNED AUTO_INCREMENT,
   `username` VARCHAR(100) NOT NULL,
   `password` VARCHAR(100) NOT NULL,
   `mail` VARCHAR(100) NOT NULL,
   PRIMARY KEY ( `id` )
)ENGINE=csdn DEFAULT CHARSET=utf8;
```

## 6. 插入数据
```sql
INSERT INTO csdn( username , password , mail) VALUES("username", "password", "admin@mail.com");
```



## 7. 数据库备份
```shell
mysqldump -u[username] -p[password] [dbname] > dbbackup.sql
```

## 8. 数据库导入
> 从一个数据库导入到另一个数据库中
```shell
mysqldump [dbname1] -u[username] -p[password] -h[host] --add-drop-table | mysql -h[host] [dbname2] -u[username] -p[password]
```

