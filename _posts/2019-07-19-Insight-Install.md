---
layout: post
title: linux系统部署Insight
categories: Security
description: Insight部署笔记
keywords: Insight, 安全管理平台
---

## 系统初始环境
- 系统环境
    - CentOS Linux release 7.5.1804
- 依赖包
    - Compatibility libraries
    - Debugging Tools
    - Development tools
### 安装基础软件
```
yum -y install bash-completion git tree lrzsz nmap vim wget sysstat net-tools
yum -y groupinstall 'development tools'
```
### 初始化配置
#### 修改主机名
```
hostnamectl set-hostname insight
```
#### 修改防火墙规则
```
firewall-cmd --zone=public --add-port=6606/tcp --permanent
firewall-cmd --zone=public --add-port=5000/tcp --permanent
firewall-cmd --zone=public --add-port=9000/tcp --permanent
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
firewall-cmd --zone=public list
```
## 环境部署
### 安装python环境
```
yum -y install python
yum -y install epel-release
yum -y install python-pip"
```
### 安装docker
```
yum -y install docker
docker --version
```
### 启动docker并设置开机自启动
```
systemctl enable docker
systemctl start docker
```
### 获取MYSQLdocker
```
docker pull mysql:5.7.13
```
### 克隆项目
```
cd /opt/ && git clone https://github.com/creditease-sec/insight.git
```
### 安装依赖库
```
yum -y install mariadb-libs.x86_64 mariadb.x86_64 libmysql mysql-devel gcc python-devel libevent-devel openldap-devel
```
### 安装依赖文件
```
cd /opt/insight && pip install -r srcpm/requirement.txt --user
```
### docker启动mysql
```
docker run -d -p 127.0.0.1:6606:3306 --name open_source_mysqldb -e MYSQL_ROOT_PASSWORD=root mysql:5.7.13
```
### 创建数据表
```
mysql -h 127.0.0.1 -P 6606 -u root -p
Enter password:root
mysql> CREATE DATABASE IF NOT EXISTS vuldb DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
mysql> grant all on vuldb.* to vuluser@'%' identified by 'vulpassword';
mysql> flush privileges;
mysql> quit
```
### 导入数据库文件
```
mysql -h127.0.0.1 -P6606 -uroot -p vuldb < srcpm/vuldb_init.sql
```
### 添加本地环境变量
```
export DEV_DATABASE_URL=mysql://vuluser:vulpassword@127.0.0.1:6606/vuldb
```
### 启动应用
```
python manage.py runserver -h 0.0.0.0 \
--link open_source_mysqldb:db \
--name open_source_srcpm \
-v $PWD/srcpm:/opt/webapp/srcpm \
-e DEV_DATABASE_URL='mysql://vuluser:vulpassword@db/vuldb' \
-e SrcPM_CONFIG=development \
-e MAIL_PASSWORD='password' \
daocloud.io/liusheng/vulpm_docker:latest \
sh -c 'supervisord -c srcpm/supervisor.conf && supervisorctl -c srcpm/supervisor.conf start all && tail -f srcpm/log/gunicorn.err && tail -f srcpm/log/mail_sender.err'
```
## 登录主页
http://xxxx:5000/srcpm/

# 备注
[Insight开源项目by宜信](https://github.com/creditease-sec/insight "Insight开源项目by宜信")