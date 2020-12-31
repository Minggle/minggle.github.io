---
layout: post
title: Kali SSH ERROR
categories: Security
description: ”服务器发送了一个意外的数据包。received:3,expected:20“问题处理
keywords: ssh, error
---

## 新安装的kali 2020.2 修改/etc/ssh/sshd_config 配置文件远程登录后，使用Xshell登录依然报错

![](https://raw.githubusercontent.com/Minggle/image/main/image/xshell_error.png)
1. 查看服务状态



```shell
sudo systemctl status sshd -l
```




![](https://raw.githubusercontent.com/Minggle/image/main/image/sshd_status_error.png) 

2. 修改sshd_config

```shell
sudo sh -c 'echo "KexAlgorithms curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group14-sha1" >> /etc/ssh/sshd_config'
```

3. 重启sshd服务

```shell
sudo systemctl restart sshd
```

解决~

