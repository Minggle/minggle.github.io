---
layout: post
title: 【转】Python模块之subprocess用法实例详解
categories: Python
description: Python模块之subprocess用法实例详解
keywords: centos, Python, subprocess
---



# 【转】Python模块之subprocess用法实例详解

# 一、简介

subprocess最早在2.4版本引入。用来生成子进程，并可以通过管道连接他们的输入/输出/错误，以及获得他们的返回值。

subprocess用来替换多个旧模块和函数：

-   os.system
-   os.spawn*
-   os.popen*
-   popen2.*
-   commands.*

运行python的时候，我们都是在创建并运行一个进程，linux中一个进程可以fork一个子进程，并让这个子进程exec另外一个程序。在python中，我们通过标准库中的subprocess包来fork一个子进程，并且运行一个外部的程序。subprocess包中定义有数个创建子进程的函数，这些函数分别以不同的方式创建子进程，所欲我们可以根据需要来从中选取一个使用。另外subprocess还提供了一些管理标准流(standard stream)和管道(pipe)的工具，从而在进程间使用文本通信。

## 二、旧有模块的使用

### 1.os.system()

执行操作系统的命令，将结果输出到屏幕，只返回命令执行状态(0：成功，非 0 ： 失败)

```javascript
import os
>>> a = os.system("df -Th")
Filesystem   Type  Size Used Avail Use% Mounted on
/dev/sda3   ext4  1.8T 436G 1.3T 26% /
tmpfs     tmpfs  16G   0  16G  0% /dev/shm
/dev/sda1   ext4  190M 118M  63M 66% /boot
>>> a
0     # 0 表示执行成功
# 执行错误的命令
>>> res = os.system("list")
sh: list: command not found
>>> res
32512    # 返回非 0 表示执行错误
```

### 2. os.popen()

执行操作系统的命令，会将结果保存在内存当中，可以用`read()`方法读取出来

```javascript
import os
>>> res = os.popen("ls -l")
# 将结果保存到内存中
>>> print res
<open file 'ls -l', mode 'r' at 0x7f02d249c390>
# 用read()读取内容
>>> print res.read()
total 267508
-rw-r--r-- 1 root root  260968 Jan 27 2016 AliIM.exe
-rw-------. 1 root root   1047 May 23 2016 anaconda-ks.cfg
-rw-r--r-- 1 root root  9130958 Nov 18 2015 apache-tomcat-8.0.28.tar.gz
-rw-r--r-- 1 root root     0 Oct 31 2016 badblocks.log
drwxr-xr-x 5 root root   4096 Jul 27 2016 certs-build
drwxr-xr-x 2 root root   4096 Jul 5 16:54 Desktop
-rw-r--r-- 1 root root   2462 Apr 20 11:50 Face_24px.ico
```

## 三、subprocess模块

### 1、subprocess.run()

```javascript
>>> import subprocess
# python 解析则传入命令的每个参数的列表
>>> subprocess.run(["df","-h"])
Filesystem      Size Used Avail Use% Mounted on
/dev/mapper/VolGroup-LogVol00
           289G  70G 204G 26% /
tmpfs         64G   0  64G  0% /dev/shm
/dev/sda1       283M  27M 241M 11% /boot
CompletedProcess(args=['df', '-h'], returncode=0)
# 需要交给Linux shell自己解析，则:传入命令字符串，shell=True
>>> subprocess.run("df -h|grep /dev/sda1",shell=True)
/dev/sda1       283M  27M 241M 11% /boot
CompletedProcess(args='df -h|grep /dev/sda1', returncode=0)
```

### 2、subprocess.call()

执行命令，返回命令的结果和执行状态，0或者非0

```javascript
>>> res = subprocess.call(["ls","-l"])
总用量 28
-rw-r--r-- 1 root root   0 6月 16 10:28 1
drwxr-xr-x 2 root root 4096 6月 22 17:48 _1748
-rw-------. 1 root root 1264 4月 28 20:51 anaconda-ks.cfg
drwxr-xr-x 2 root root 4096 5月 25 14:45 monitor
-rw-r--r-- 1 root root 13160 5月  9 13:36 npm-debug.log
# 命令执行状态
>>> res
0
```

### 3、subprocess.check_call()

执行命令，返回结果和状态，正常为0 ，执行错误则抛出异常

```javascript
>>> subprocess.check_call(["ls","-l"])
总用量 28
-rw-r--r-- 1 root root   0 6月 16 10:28 1
drwxr-xr-x 2 root root 4096 6月 22 17:48 _1748
-rw-------. 1 root root 1264 4月 28 20:51 anaconda-ks.cfg
drwxr-xr-x 2 root root 4096 5月 25 14:45 monitor
-rw-r--r-- 1 root root 13160 5月  9 13:36 npm-debug.log
0
>>> subprocess.check_call(["lm","-l"])
Traceback (most recent call last):
 File "<stdin>", line 1, in <module>
 File "/usr/lib64/python2.7/subprocess.py", line 537, in check_call
  retcode = call(*popenargs, **kwargs)
 File "/usr/lib64/python2.7/subprocess.py", line 524, in call
  return Popen(*popenargs, **kwargs).wait()
 File "/usr/lib64/python2.7/subprocess.py", line 711, in __init__
  errread, errwrite)
 File "/usr/lib64/python2.7/subprocess.py", line 1327, in _execute_child
  raise child_exception
OSError: [Errno 2] No such file or directory
```

### 4、subprocess.getstatusoutput()

接受字符串形式的命令，返回 一个元组形式的结果，第一个元素是命令执行状态，第二个为执行结果

```javascript
#执行正确
>>> subprocess.getstatusoutput('pwd')
(0, '/root')
#执行错误
>>> subprocess.getstatusoutput('pd')
(127, '/bin/sh: pd: command not found')
```

### 5、subprocess.getoutput()

接受字符串形式的命令，放回执行结果

```javascript
>>> subprocess.getoutput('pwd')
'/root'
```

### 6、subprocess.check_output()

执行命令，返回执行的结果，而不是打印

```javascript
>>> res = subprocess.check_output("pwd")
>>> res
b'/root\n' # 结果以字节形式返回
```

## 四、subprocess.Popen()

其实以上subprocess使用的方法，都是对subprocess.Popen的封装，下面我们就来看看这个Popen方法。

### 1、stdout

标准输出

```javascript
>>> res = subprocess.Popen("ls /tmp/yum.log", shell=True, stdout=subprocess.PIPE) # 使用管道
>>> res.stdout.read()  # 标准输出
b'/tmp/yum.log\n'
res.stdout.close()  # 关闭
```

### 2、stderr

标准错误

```javascript
>>> import subprocess
>>> res = subprocess.Popen("lm -l",shell=True,stdout=subprocess.PIPE,stderr=subprocess.PIPE)
# 标准输出为空
>>> res.stdout.read()
b''
#标准错误中有错误信息
>>> res.stderr.read()
b'/bin/sh: lm: command not found\n'
```

注意：上面的提到的标准输出都为啥都需要等于subprocess.PIPE,这个又是啥呢？原来这个是一个管道，这个需要画一个图来解释一下：

### 4、poll()

定时检查命令有没有执行完毕，执行完毕后返回执行结果的状态，没有执行完毕返回None

```javascript
>>> res = subprocess.Popen("sleep 10;echo 'hello'",shell=True,stdout=subprocess.PIPE,stderr=subprocess.PIPE)
>>> print(res.poll())
None
>>> print(res.poll())
None
>>> print(res.poll())
0
```

### 5、wait()

等待命令执行完成，并且返回结果状态

```javascript
>>> obj = subprocess.Popen("sleep 10;echo 'hello'",shell=True,stdout=subprocess.PIPE,stderr=subprocess.PIPE)
>>> obj.wait()
# 中间会一直等待
0
```

### 6、terminate()

结束进程

```javascript
import subprocess
>>> res = subprocess.Popen("sleep 20;echo 'hello'",shell=True,stdout=subprocess.PIPE,stderr=subprocess.PIPE)
>>> res.terminate() # 结束进程
>>> res.stdout.read()
b''
```

### 7、pid

获取当前执行子shell的程序的进程号

```javascript
import subprocess
>>> res = subprocess.Popen("sleep 5;echo 'hello'",shell=True,stdout=subprocess.PIPE,stderr=subprocess.PIPE)
>>> res.pid # 获取这个linux shell 的 进程号
2778
```