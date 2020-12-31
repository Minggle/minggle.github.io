---
layout: post
title: CentOS 7 安装 Proxychain-NG
categories: CentOS
description: CentOS 7 安装 Proxychain-NG
keywords: centos, Proxychain
---

> 由于种种原因，很多命令下载速度很慢而且不稳定，所以使用proxychains进行代理，代理服务器自备。


## CentOS7 安装 ProxyChains-NG

1. 从GitHub下载源码
```shell
cd /usr/local/src/
git clone https://github.com/rofl0r/proxychains-ng
```
2. 安装编译环境，编译安装
```shell
yum -y install gcc
cd proxychains-ng/
./configure --prefix=/usr --sysconfdir=/etc
make
make install
make install-config
```
3. 配置代理config
```shell
vim /etc/proxychains.conf
##添加代理服务到配置文件最下面
[ProxyList]
#protocol	proxy_ip	proxy_port
socks4  127.0.0.1 7777
```

4. proxy使用
```shell
proxychains4 「command」
eg.
proxychains4 curl www.google.com
proxychains4 pip install xxxxxxx
proxychains4 docker pull xxxxxxx
proxychains4 git clond xxxxxx
```
