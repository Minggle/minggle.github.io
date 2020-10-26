---
layout: post
title: CentOS opehssh 7.5 更新
categories: CentOS
description: CentOS opehssh 7.5 更新
keywords: centos, opehssh, 7.5, 更新
---

# 0x00 升级前准备

## 确认ssh版本

```shell
ssh -V
```

## 安装升级依赖

```shell
yum -y install gcc openssl-devel zlib-devel
```

## 安装telnet
- 确认telnet是否安装

```shell
rpm -qa | grep telnet
```
- 已安装，则测试是否可以通过telnet登陆
- 未安装则进行安装，设置开机自启动，放开防火墙配置

```shell
yum -y install telnet-server* telnet
vi /etc/xinetd.d/telnet
		disable         = no
mv /etc/securetty /etc/securetty.old
service xinetd start
chkconfig xinetd on
```

- iptables配置（Centos6.x）

```shell
iptables -I INPUT -p tcp --dport 23 -jACCEPT
iptables -I INPUT -p udp --dport 23 -jACCEPT
service iptables save
service iptables restart
```
- firewall配置(CentOS7.x)

```shell
firewall-cmd --zone=public --add-port=23/tcp --permanent
firewall-cmd --zone=public --add-port=23/udp --permanent
firewall-cmd --reload#刷新防火墙规则
firewall-cmd --zone=public --list-port#查看所有开放端口
```

# 0x01 升级过程
## 上传源码包到/usr/local/src/
- 方式1：sftp

```shell
cd /usr/local/src/
put d:/openssh-7.5p1.tar.gz
```
- 方式2：lrzsz

```shell
cd /usr/local/src/
yum -y install lrzsz
rz -y
```
## 备份原有ssh配置
```shell
mv /etc/ssh /etc/ssh.old
mv /etc/init.d/sshd /etc/init.d/sshd.old
```
## 卸载openssh

```shell
rpm -qa | grep openssh  ## 查询安装的包
rpm -qa | grep openssh | xrags rpm -e --noscripts
rpm -qa | grep openssh | xrags rpm -e --nodeps
rpm -qa | grep openssh  ## 查询卸载结果，确认已经卸载完成
##删除不完全时使用
rpm -e --nodeps {包名}
```
## 其他配置调整

```shell
install -v -m700 -d /var/lib/sshd
chown -v root:sys /var/lib/sshd
groupadd -g 50 sshd
useradd -c 'sshd PrivSep' -d /var/lib/sshd -g sshd -s /bin/false -u 50 sshd
```
- 调整selinux配置

```shell
vim /etc/selinux/config
    SELINUX=disabled
```

## 编译安装openssh

```shell
tar -zxvf openssh-7.5p1.tar.gz && cd openssh-7.5p1
./configure --prefix=/usr --sysconfdir=/etc/ssh --with-md5-passwords  --with-zlib --with-openssl-includes=/usr --with-privsep-path=/var/lib/sshd
make && make install
```

- 其他配置调整

```shell
install -v -m755    contrib/ssh-copy-id /usr/bin
install -v -m644    contrib/ssh-copy-id.1 /usr/share/man/man1
install -v -m755 -d /usr/share/doc/openssh-7.5p1
install -v -m644    INSTALL LICENCE OVERVIEW README* /usr/share/doc/openssh-7.5p1
```

- ssh配置调整

```shell
echo 'X11Forwarding yes' >> /etc/ssh/sshd_config
echo "PermitRootLogin yes" >> /etc/ssh/sshd_config  
cp -p contrib/redhat/sshd.init /etc/init.d/sshd
chmod +x /etc/init.d/sshd
chkconfig  --add  sshd
chkconfig  sshd  on
chkconfig  --list  sshd
touch /etc/ssh/ssh_host_key.pub
service sshd restart
ssh -V
```

## 验证登陆没问题后关闭telnet服务

```shell
mv /etc/securetty.old /etc/securetty
chkconfig  xinetd off
service xinetd stop
```


## 如需还原之前的ssh配置信息，可直接删除升级后的配置信息，恢复备份。

```shell
# rm -rf /etc/ssh
# mv /etc/ssh.old /etc/ssh
```



# 0x02 附录
## 报错
```shell
checking OpenSSL library version... configure: error: OpenSSL >= 1.0.1 required (have "10000003 (OpenSSL 1.0.0-fips 29 Mar 2010)")
yum remove openssl -y
yum install openssl
```

## 升级history

```shell
  440  ssh -V
  441  yum install gcc openssl-devel zlib-devel
  442  rpm -qa | grep gcc
  443  rpm -qa | grep ssl
  444  rpm -qa | grep zlib
  445  cd /usr/local/src/
  446  rpm -qa | grep telnet
  447  yum -y install telnet-server* telnet
  448  vi /etc/xinetd.d/telnet
  449  mv /etc/securetty /etc/securetty.old
  450  service xinetd start
  451  chkconfig xinetd on
  452  mv /etc/ssh /etc/ssh.old
  453  mv /etc/init.d/sshd /etc/init.d/sshd.old
  454  rpm -qa | grep openssh
  455  yum remove openssh -y
  456  rpm -qa | grep openssh
  457  rpm -e --noscripts openssh-server-5.3p1-94.el6.x86_64
  458  rpm -qa | grep openssh
  459  install -v -m700 -d /var/lib/sshd
  460  chown -v root:sys /var/lib/sshd
  461  groupadd -g 50 sshd
  462  useradd -c 'sshd PrivSep' -d /var/lib/sshd -g sshd -s /bin/false -u 50 sshd
  463  tar -zxvf openssh-7.5p1.tar.gz
  464  cd openssh-7.5p1
  465  ./configure --prefix=/usr --sysconfdir=/etc/ssh --with-md5-passwords  --with-zlib --with-openssl-includes=/usr --with-privsep-path=/var/lib/sshd
  466  yum remove openssl -y
  467  yum install openssl -y
  468  ./configure --prefix=/usr --sysconfdir=/etc/ssh --with-md5-passwords  --with-zlib --with-openssl-includes=/usr --with-privsep-path=/var/lib/sshd
  469  make && make install
  470  install -v -m755    contrib/ssh-copy-id /usr/bin
  471  install -v -m644    contrib/ssh-copy-id.1 /usr/share/man/man1
  472  install -v -m755 -d /usr/share/doc/openssh-7.5p1
  473  install -v -m644    INSTALL LICENCE OVERVIEW README* /usr/share/doc/openssh-7.5p1
  474  echo 'X11Forwarding yes' >> /etc/ssh/sshd_config
  475  echo "PermitRootLogin yes" >> /etc/ssh/sshd_config   #允许root用户通过ssh登录
  476  cp -p contrib/redhat/sshd.init /etc/init.d/sshd
  477  chmod +x /etc/init.d/sshd
  478  chkconfig  --add  sshd
  479  chkconfig  sshd  on
  480  chkconfig  --list  sshd
  481  touch /etc/ssh/ssh_host_key.pub
  482  ssh -V
  483  service sshd restart
  484  clear
  485  service sshd status
  486  ps -ef | grep ssh
  487  exit
  488  ssh -V
  489  ps -ef | grep ssh
  490  mv /etc/securetty.old /etc/securetty
  491  chkconfig  xinetd off
  492  service xinetd stop
  493  vi /etc/selinux/config
  494  history 
```