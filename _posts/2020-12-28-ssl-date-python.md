---
layout: post
title: Python监控SSL证书有效期脚本 
categories: Python
description: Python监控SSL证书有效期脚本 
keywords: centos, Python, SSL, 有效期
---
# ssl证书有效期监控脚本

>   需求：
>
>   SSL证书有效期监控，过期前一个月邮件提醒。



## 1. 环境准备

基础环境为`CentOS Linux release 7.6.1810 (Core)`



### 1.1 操作系统基础依赖安装

```shell
yum install yum-utils wget vim libcurl-devel
```



### 1.2 Python 3.9.1 安装

```shell
# 重度强迫症患者，只用最新版，下载3.9.1！
mkdir -p /opt/python/python3.9.1 && cd /usr/local/src/ 
wget https://www.python.org/ftp/python/3.9.1/Python-3.9.1.tar.xz && tar Jxvf Python-3.9.1.tar.xz
cd Python-3.9.1/
./configure --prefix=/opt/python/python3.9.1 --with-ssl
make && make install

# 创建软链接
rm -rf /usr/bin/python
rm -rf /usr/bin/pip
ln -s /opt/python/python3.9.1/bin/python3.9 /usr/bin/python3.9
ln -s /usr/bin/python3.9 /usr/bin/python3
ln -s /usr/bin/python3 /usr/bin/python
ln -s /opt/python/python3.9.1/bin/pip3.9 /usr/bin/pip3.9
ln -s /usr/bin/pip3.9 /usr/bin/pip3
ln -s /usr/bin/pip3 /usr/bin/pip

# 修改python软链后需要修改部分命令环境变量
sed 's|/usr/bin/python|/usr/bin/python2.7|g' /usr/bin/yum /usr/libexec/urlgrabber-ext-down -i
```



### 1.3 Mariadb 安装

```shell
curl -sS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash
proxychains4 yum makecache
proxychains4 yum -y install mariadb mariadb-server mariadb-devel
systemctl enable mariadb
systemctl start mariadb
mysql_secure_installation
mysql -uroot -p
```



## 2. 开始部署



### 2.1 数据库初始化

```mysql
CREATE DATABASE `ssldb` CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER 'sslscan'@'%' IDENTIFIED BY 'dbpassword';
grant all privileges  on ssldb.* to 'sslscan'@'%';
flush privileges;

CREATE TABLE `ssl_table` (
`ID`  int(11) NOT NULL AUTO_INCREMENT ,
`check_time`  text NULL ,
`domain`  text NULL ,
`s_time`  text NULL ,
`e_time`  text NULL ,
`c_info`  text NULL ,
`if_s`  text NULL ,
PRIMARY KEY (`ID`)
)
;
```



### 2.2 安装 Python 依赖库

```shell
# 安装依赖库
pip install pyOpenSSL pymysql pycurl
```



### 2.3 脚本文件


-   脚本路径：`/usr/local/src/python/ssl_writer.py`
-   证书路径：`/usr/local/src/ssl/`


```python
#!/use/bin/python
# -*- coding:utf-8 -*-
# 依赖库pyOpenSSL pymysql 
from OpenSSL import crypto
import time
import os
import pymysql
import datetime
import smtplib
from email.mime.text import MIMEText
from email.header import Header

# 定义「.pen」文件写入列表
def file_filter(f):
    if f[-4:] == '.pem':
        return 1
    else:
        return 0

# 定义邮件发送告警
def sed_mail(domain, date):
    # 第三方 SMTP 服务
    mail_host = "stmp.163.com"  # 设置服务器
    mail_user = "user"  # 用户名
    mail_pass = "passwd"  # 口令

    sender = 'send@163.com'
    receivers = ['ccc@163.com',
                 # 'bbb@163.com',
                 # 'aaa@163.com'
                 ]  # 接收邮件，可设置为你的163邮箱或者其他邮箱

    message = MIMEText("域名： "+domain+" 的 SSL 证书有效期还剩余 "+date+" 天，即将过期，请及时续签！", 'plain', 'utf-8')
    message['From'] = Header("xx SSL 证书有效期监控提醒", 'utf-8')
    message['To'] = Header("管理员", 'utf-8')

    subject = 'SSL 证书有效期监控提醒'
    message['Subject'] = Header(subject, 'utf-8')

    try:
        # 如25端口，则使用如下：
        '''smtpObj = smtplib.SMTP()
        smtpObj.connect(mail_host, 25)  # 25 为 SMTP 端口号'''
        # 修改为SSL SMTP服务器
        smtpObj = smtplib.SMTP_SSL(mail_host)
        smtpObj.login(mail_user, mail_pass)
        smtpObj.sendmail(sender, receivers, message.as_string())
        print(time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())+" 域名： "+domain+" SSL 证书有效期告警邮件发送成功！")
    except smtplib.SMTPException:
        print(time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())+" 域名： "+domain+" Error: 无法发送邮件！")


# 数据库连接设置
db = pymysql.connect(
    "127.0.0.1",
    "sslscan",	# 数据库账户
    "dbpassword", # 数据库密码
    "ssldb"
)

cursor = db.cursor()
cursor.execute("SET NAMES utf8"); # 设置数据库连接字符集

# 重新初始化数据库
sql_clean = "truncate table ssl_table;"
cursor.execute(sql_clean)
db.commit()

# pem文件名写入files列表,path路径根据需要修改

path = '/usr/local/src/ssl/'
dirs = os.listdir(path)
files = list(filter(file_filter, dirs))


# 结果输出
for i in range(len(files)):
    # print(file)
    cert_file = path+files[i]
    # print(cert_file)
    cert = crypto.load_certificate(crypto.FILETYPE_PEM, open(cert_file).read())
    subject = cert.get_subject()

    # 获取查询时间
    check_time = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())
    # print(check_time)

    # 得到证书的域名
    domain = subject.CN
    # print(domain)

    # 获取证书开始时间
    start_time = cert.get_notBefore().decode('utf-8')
    s_time = start_time[0:4]+"-"+start_time[4:6]+"-"+start_time[6:8]+" "+start_time[8:10]+":"+start_time[10:12]+":"+start_time[12:14]
    # print(s_time)

    # 获取证书到期时间
    end_time = cert.get_notAfter().decode('utf-8')
    e_time = end_time[0:4]+"-"+end_time[4:6]+"-"+end_time[6:8]+" "+end_time[8:10]+":"+end_time[10:12]+":"+end_time[12:14]
    # print(e_time)

    # 得到证书颁发信息
    issuer = cert.get_issuer()
    # print(issuer)
    issued_cn = issuer.CN
    # print(issued_cn)
    issued_ou = issuer.OU
    # print(issued_ou)
    issued_o = issuer.O
    # print(issued_o)
    issued_c = issuer.C
    # print(issued_c)
    c_info = "CN=" + issued_cn + ",OU=" + issued_ou + ",O=" + issued_o + ",C=" + issued_c
    # print(c_info)

    # 写入数据库
    sql_reinto = 'INSERT ssl_table SET domain = "'+domain+'", check_time = "'+check_time+'", s_time = "'+s_time+'", e_time = "'+e_time+'", c_info = "'+c_info+'", ID = '+str(i+1) #格式化时间及证书颁发机构名称，构造SQL查询
    cursor.execute(sql_reinto)
    db.commit()
    # time.sleep(1) # 睡一会儿，免得太High

    # 确认时间，并发送到邮件服务器。
    sql_select = 'select check_time,e_time from ssl_table where ID = '+str(i+1)
    cursor.execute(sql_select)
    results = cursor.fetchall()
    # print(results)

    for row in results:
        check_time = row[0]
        check_date = check_time[0:10]
        date1 = time.strptime(check_date, "%Y-%m-%d")
        check_date_number = datetime.datetime(date1[0],date1[1],date1[2])
        end_time = row[1]
        end_date = end_time[0:10]
        date2=time.strptime(end_date,"%Y-%m-%d")
        end_date_number = datetime.datetime(date2[0],date2[1],date2[2])
        data_check = end_date_number - check_date_number
        if data_check.days < 30:
            # print("域名： "+domain+" 的 SSL 证书有效期还剩余 "+str(data_check.days)+" 天，即将过期，请及时续签！")
            sed_mail(domain, str(data_check.days))
        else:
            print(time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())+" 域名： "+domain+" 的 SSL 证书有效期还剩余 "+str(data_check.days)+" 天，无需提醒！")
db.close()

```



## 3. 定时脚本任务

```shell
# 每天早上8点进行定时任务
echo "0 8 * * * /usr/bin/python /usr/local/src/python/ssl_writer.py  >> /usr/local/src/ssl/var.log" >> /var/spool/cron/root

```

