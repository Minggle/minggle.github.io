---
layout: post
title: CentOS 7 本地YUM仓库搭建
categories: CentOS
description: CentOS 7 本地YUM仓库搭建
keywords: centos, yum
---

# 本地YUM仓库搭建
| 序号 | 修改人  | 版本 | 时间      | 备注 |
| ---- | ------- | ---- | --------- | ---- |
| 1.   | Minggle | v1.0 | 2019.6.28 | 新建 |

# 一、Server端配置

## 0x00 设置基础配置
```
# 设置主机名
hostnamectl set-hostname yum

# 配置防火墙
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --reload
```

## 0x01 获取阿里云镜像repo
```
# 获取epel源
wget -O /etc/yum.repos.d/epel.repo https://mirrors.aliyun.com/repo/epel-7.repo
# 获取base源
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

```
## 0x02 配置http服务

```
# 安装httpd
yum install httpd yum-utils -y
# 调整SeLinux配置
sed 's|enforcing|disabled|g' /etc/selinux/config -i
reboot
# 设置开机自启动
systemctl enable httpd
# 启动httpd
systemctl start httpd
```

## 0x03 同步镜像包到本地
```
# 查看yum仓库标识
yum repolist
# 使用epel作为本地yum源，用/var/www/html作为yum仓库根目录
reposync -r epel -p /var/www/html
# 使用base作为本地yum源，用/var/www/html作为yum仓库根目录
reposync -r base -p /var/www/html
```


## 0x04 生成目录索引
```
#第一个目录是repodata存放目录，第二个目录是需要生成索引信息yum源仓库目录
createrepo -pdo /var/www/html/epel/ /var/www/html/epel/
createrepo -pdo /var/www/html/base/ /var/www/html/base/

```

## 0x05 配置自动同步更新脚本
1. epel 脚本
```
# vim /root/yum-update-epel.sh

#!/bin/bash
datetime=`date +"%Y-%m-%d"`
exec > /var/log/epel.log　　#同步日志输出
reposync -d -r epel -p /var/www/html/  　　#同步镜像源
if [ $? -eq 0 ];then
    createrepo --update  /var/www/html/epel 　　#每次添加新的rpm时,必须更新epel索引信息
echo "SUCESS: $datetime epel update successful"
else
echo "ERROR: $datetime epel update failed"
fi

```

2. base 脚本
```
# vim /root/yum-update-base.sh

#!/bin/bash
datetime=`date +"%Y-%m-%d"`
exec > /var/log/base.log　　#同步日志输出
reposync -d -r base -p /var/www/html/  　　#同步镜像源
if [ $? -eq 0 ];then
    createrepo --update  /var/www/html/base 　　#每次添加新的rpm时,必须更新epel索引信息
echo "SUCESS: $datetime base update successful"
else
echo "ERROR: $datetime base update failed"
fi
```

3. 定时任务
```
# 每周六凌晨三点启动更新脚本
# crontab -e
* 3 * * 6 /usr/local/src/yum-update-epel.sh
* 3 * * 6 /usr/local/src/yum-update-base.sh
systemctl restart crond
systemctl enable crond
```


# 二、Client端配置
## 0x00 调整epel更新源
```
# vim /etc/yum.repos.d/epel-7.repo

[epel]
name=local epel
baseurl=http://192.168.106.111/epel
enabled=1
gpgcheck=0 
```

## 0x01 调整base更新源
```
# vim /etc/yum.repos.d/CentOS-Base.repo

[base]
name=local base
baseurl=http://192.168.106.111/base
enabled=1
gpgcheck=0
# 注释掉[base]项目中其他内容
```

## 0x02 测试与实施

```
yum clean all
# mv /var/cache/yum /tmp/
yum makecache
yum repolist
yum -y install xxx
```


# 三、备注
1. 配置放开客户端服务器到Server:80端口通信。
2. 配置放开Server到互联网mirrors源通信。