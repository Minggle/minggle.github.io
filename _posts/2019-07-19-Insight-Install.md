---
layout: post
title: linux系统部署Insight
categories: Sceurity
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

## 安装基础软件

```
yum -y install git tree lrzsz nmap vim wget zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make bash-completion
yum groupinstall "Development tools"
```
### 修改hostname
```
hostnamectl set-hostname semf
logout
```
### 部署python
```
cd /usr/local/src/ && wget https://www.python.org/ftp/python/3.6.5/Python-3.6.5.tar.xz
tar -xvJf Python-3.6.5.tar.xz
cd Python-3.6.5/
./configure prefix=/usr/local/python3
make && make install
ln -s /usr/local/python3/bin/python3 /usr/bin/python3
```
### 部署erlang
```
cd /usr/local/src/ && wget http://www.rabbitmq.com/releases/erlang/erlang-19.0.4-1.el7.centos.x86_64.rpm
rpm -ivh erlang-19.0.4-1.el7.centos.x86_64.rpm
yum -y install erlang
erl -version
```
### 部署rabbitmq
```
cd /usr/local/src/ && wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.9/rabbitmq-server-3.6.9-1.el7.noarch.rpm
yum install rabbitmq-server-3.6.9-1.el7.noarch.rpm -y
```
### 启动服务
```
systemctl start rabbitmq-server#启动rabbitmq服务
rabbitmq-plugins enable rabbitmq_management#开启WEB端
rabbitmqctl add_user <user> <password>#添加用户
rabbitmqctl add_vhost semf#添加vhost
rabbitmqctl set_user_tags <user> administrator#设置权限
rabbitmqctl set_permissions -p semf root ".*" ".*" ".*"#正则全部权限
```
### 防火墙设置
```
firewall-cmd --zone=public --add-port=5672/tcp --permanent#开启rabbitmq api端口
firewall-cmd --zone=public --add-port=15672/tcp --permanent#开启rabbitmq web端口
firewall-cmd --zone=public --add-port=8000/tcp --permanent#开启8000端口
firewall-cmd --zone=public --add-port=80/tcp --permanent#开启80端口
firewall-cmd --reload#刷新防火墙规则
firewall-cmd --zone=public --list-port#查看所有开放端口
```
## 安装应用
### 克隆项目
```
cd /opt/ && git clone https://gitee.com/gy071089/SecurityManageFramwork.git
```
### 修改配置文件
```
cd /opt/SecurityManageFramwork/SeMF && vim settings.py
#设置网站根地址
WEB_URL = 'http://x.x.x.x:8000'
#设置邮箱
#设置邮箱
EMAIL_HOST = 'smtp.163.com'          #SMTP地址
EMAIL_PORT = 25                 #SMTP端口
EMAIL_HOST_USER = 'xxxxxxxx@163.com'    #我自己的邮箱
EMAIL_HOST_PASSWORD = 'xxxxxxxx'         #我的邮箱密码
EMAIL_SUBJECT_PREFIX = u'[SeMF]'      #为邮件Subject-line前缀,默认是'[django]'
EMAIL_USE_TLS = True               #与SMTP服务器通信时，是否启动TLS链接(安全链接)。默认是false
#管理员站点
SERVER_EMAIL = 'xxxxxxxx@163.com'
DEFAULT_FROM_EMAIL = '安全管控平台xxxxxxxx@163.com>'
#设置队列存储
BROKER_URL = 'amqp://root:toor@semf/semf'    #设置与rabbitmq一致
CELERY_ACCEPT_CONTENT = ['pickle', 'json', 'msgpack', 'yaml']
```
### 初始化安装配置
```
python3 -m pip install -r requirements.txt#安装python库
python3 manage.py makemigrations#生成数据表
python3 manage.py migrate
python3 manage.py createsuperuser#创建超级管理员
Username (leave blank to use 'root'):
Email address: xxx@xxx.cn
Password: 
Password (again): 
Superuser created successfully.
```
### 数据库初始化
```
python3 cnvd_xml.py #初始化数据库，主要生成角色，权限等信息
python3 initdata.py #用于同步cnvd漏洞数据文件，文件位于cnvd_xml目录下，可自行调整，该文件夹每周更新一次，
```
### 创建异步任务脚本
```
cat >>celery.sh<<EOF
python3 -m celery -A SeMF worker -l info --autoscale=10,4 >> /opt/SeMF/SecurityManageFramwork/logs/crlery.log 2>&1 &
echo 'Start celery for semf'
EOF
chmod u+x celery.sh 
sudo sh celery.sh #执行异步任务
ps -ef | grep celery
ps -ef | grep celery | grep -v grep | awk '{print $2}' | xargs kill -9
ps -ef | grep celery
python3 manage.py runserver 0.0.0.0:8000#测试服务是否正常
```
### 创建启动服务脚本
```
cat >>runsemf.sh<<EOF
python3 /opt/SeMF/SecurityManageFramwork/manage.py runserver 0.0.0.0:8000 >> /opt/SeMF/SecurityManageFramwork/logs/semflog.log 2>&1 &
echo 'Start SEMF'
EOF
chmod u+x runsemf.sh 
sudo sh celery.sh 
```
### 优化supervisor守护进程
```
yum install epel-release
yum install -y supervisor
echo \"files = /opt/SeMF/SecurityManageFramwork/conf/*.conf\" >> /etc/supervisord.conf
mkdir /opt/SeMF/SecurityManageFramwork/conf
touch /opt/SeMF/SecurityManageFramwork/conf/celery.conf
touch /opt/SeMF/SecurityManageFramwork/conf/semf.conf
```
```
cat >>conf/semf.conf<<EOF
[program:celery]
command=python3 -m celery -A SeMF worker -l info --autoscale=10,4     ; supervisor启动命令
directory=/opt/SeMF/SecurityManageFramwork/                                                 ; 项目的文件夹路径
startsecs=10                                                                       ; 启动时间
stopwaitsecs=60                                                                          ; 终止等待时间
autostart=true                                                                         ; 是否自动启动
autorestart=true                                                                       ; 是否自动重启
stopasgroup=true
stdout_logfile=/opt/SeMF/SecurityManageFramwork/logs/celerylog.log                           ; log 日志
stderr_logfile=/opt/SeMF/SecurityManageFramwork/logs/celerylog.err                           ; 错误日志
EOF
```
```
cat >>runsemf.sh<<EOF
[program:semf]
command=python3 manage.py runserver 0.0.0.0:8000     ; supervisor启动命令
directory=/opt/SeMF/SecurityManageFramwork/                                                 ; 项目的文件夹路径
startsecs=10                                                                       ; 启动时间
stopwaitsecs=60                                                                          ; 终止等待时间
autostart=true                                                                         ; 是否自动启动
autorestart=true                                                                       ; 是否自动重启
stdout_logfile=/opt/SeMF/SecurityManageFramwork/logs/semflog.log                           ; log 日志
stderr_logfile=/opt/SeMF/SecurityManageFramwork/logs/semflog.err                           ; 错误日志
EOF
```
```
supervisord -c /etc/supervisord.conf
supervisorctl reload
supervisorctl start all
```
### 外部地址访问
```
http://xxxx:8000
http://xxxx.8000/semf#管理页面"
# 备注
[SeMF开源项目by残源](https://gitee.com/gy071089/SecurityManageFramwork "SeMF开源项目by残源")
```