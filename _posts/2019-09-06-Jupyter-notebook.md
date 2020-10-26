---
layout: post
title: 利用Jupyter在linux部署个人云笔记
categories: Python
description: 利用Jupyter在linux部署个人云笔记
keywords: Jupyter, 云笔记
---

# 利用Jupyter在linux部署个人云笔记

- 部署环境
	- 系统环境
		- CentOS Linux release 7.5.1804
	- 依赖包
		- Compatibility libraries
		- Debugging Tools
		- Development tools
- 安装所需软件
`yum -y install wget bzip2`
- 获取安装包并安装
```shell
cd /usr/local/src/
wget https://repo.anaconda.com/archive/Anaconda3-5.2.0-Linux-x86_64.sh
chmod +x Anaconda3-5.2.0-Linux-x86_64.sh
sh Anaconda3-5.2.0-Linux-x86_64.sh #安装目录可修改（修改为/opt/anaconda3），环境变量选择写入~/.bashrc
```
- 使环境变量生效
`source ~/.bashrc`
- 安装运行插件
```shell
pip install msgpack
conda install -c anaconda msgpack-python
pip install jupyter_contrib_nbextensions
jupyter contrib nbextension install --user
```
- 设置密码
```shell
ipython
In [1]: from notebook.auth import passwd
In [2]: passwd() #密码sha1需要copy下来，下面配置需要用到
```
- 编辑配置文件
```shell
jupyter notebook --generate-config --allow-root
vim /root/.jupyter/jupyter_notebook_config.py
#c.NotebookApp.ip = '127.0.0.1'
#c.NotebookApp.port = 9527
#c.NotebookApp.notebook_dir = u'/root/jupyter_dir'
#c.NotebookApp.open_browser = False
#c.NotebookApp.password = u'sha1:49f6ad7d12ee:ef70177be7ca8ba575f'#此sha1从上步骤取得。
```
- 编写启动脚本  #可优化
```shell
cat >>/root/jupyter_run.sh<<EOF
jupyter notebook --allow-root >> /opt/anaconda3/logs/jupyter_log.log 2>&1 &
echo "start jupyter"
EOF
mkdir -p /opt/anaconda3/logs
cd ~ && chmod +x jupyter_run.sh && mkdir jupyter_dir
sh jupyter_run.sh
```
- 放行防火墙端口
```shell
firewall-cmd --zone=public --add-port=9527/tcp --permanent
firewall-cmd --reload
```
- 外网访问测试"
http://xxxx:9527

## 配置supervisor守护jupyter进程
- 安装supervisor
- 编写配置文件
- 启动supervisor
```shell
yum -y install supervisor
echo_supervisord_conf > /etc/supervisord.conf
echo "files = /root/jupyter/conf/*.conf" >>/etc/supervisord.conf
```
-  `vim /root/jupyter/conf/jupyter.conf`
```shell
[program:jupyter]
command=jupyter notebook --allow-root							; supervisor启动命令
directory=/root/												 ; 项目的文件夹路径
startsecs=10													 ; 启动时间
stopwaitsecs=60												  ; 终止等待时间
autostart=true												   ; 是否自动启动
autorestart=true												 ; 是否自动重启
stdout_logfile=/root/jupyter/logs/log.log						; log 日志
stderr_logfile=/root/jupyter/logs/log.err						; 错误日志
```
- 启动supervisor守护进程
```shell
supervisord -c /etc/supervisord.conf
supervisorctl reload
supervisorctl start all
```

## 使用caddy代理
- 安装caddy
- 编写Caddyfile
- 启动caddy
```shell
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/caddy_install.sh && chmod +x caddy_install.sh
```
-  `vim /usr/local/caddy/Caddyfile`
```shell
note.mingsec.com 
{
  gzip
  tls xxx@163.com
  log /usr/local/caddy/caddy.log
  proxy / 127.0.0.1:9527 
	{
	header_upstream X-Real-IP  {remote}
	header_upstream Host {host}
	websocket
	}
	rewrite 
	{
		r "~* /(api/kernels/[^/]+/(channels|iopub|shell|stdin)|terminals/websocket)/?"
		to /proxy/{uri}
	}
}
/etc/init.d/caddy restart"
```

