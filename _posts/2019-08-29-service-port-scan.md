---
layout: post
title: 系统服务端口监测脚本
categories: Scan
description: 系统服务端口监测脚本
keywords: 服务, 监测, 脚本
---

- 每半小时扫描一次系统服务端口，异常则通过短信api告警
- 基础环境 Centos 7.5
- nmap安装

```shell
yum -y install nmap
```
- scan.sh

```shell
#!/bin/bash
source /etc/profile
source /root/.bashrc

#添加定时任务
#conrtab -e
#0,30 * * * * /bin/sh /usr/local/src/shell/scan.sh > /dev/null 2>&1

#LIST数组中添加"ip:port:name"
LIST=("10.252.222.1:8080:erp" "10.252.222.1:8081:app")
#定义时间变量
DATE_now=`date -R | awk '{print $2","$3","$4","$5}'`
#定义i做LIST自加
i=0
#定义scan函数，正常输出日志，异常输出IP：PORT到cheaklist数组
function scan(){
	CHEAK=`nmap $1 -p $2 -Pn | grep -v nmap | grep $2 | awk '{print $2}'`
	if [ $CHEAK = "open" ];then
		NOTE="日志："$DATE_now",目前"$1"系统"$2"端口巡检正常。"
		LOG=` echo -e $NOTE >> /usr/local/src/shell/sysportlog.log`
#		echo $NOTE	  #测试日志打印
		$LOG
	else
#		echo $i
		cheaklist[$i]="$1":"$2"
		((i++))
#		echo ${cheaklist[@]}
	fi
}

#定义rescan函数，针对cheaklist数组进行再次监测函数
function rescan(){
		CHEAK=`nmap $1 -p $2 -Pn | grep -v nmap | grep $2 | awk '{print $2}'`
		if [ $CHEAK = "open" ];then
			NOTE="日志："$DATE_now",目前"$1"系统"$2"端口巡检正常。"
			LOG=` echo -e $NOTE >> /usr/local/src/shell/sysportlog.log`
#			echo $NOTE	#测试日志打印
			$LOG
		else
			NOTE_err="告警："$DATE_now",目前"$1"系统"$2"端口巡检异常，请排查应用服务是否正常。"
			LOG_err=`echo -e $NOTE_err >> /usr/local/src/shell/sysporterr.log`
#			echo $NOTE_err  #测试日志打印
			$LOG_err
			curl http://192.168.91.4:9999/Sunriver/notice  -X POST -H "Content-Type:application/json" -d '{"ip":"'$1'","content":"'$NOTE_err'"}'
		fi
}



for list in ${LIST[@]}
do
#	echo $list	#测试日志打印
	IP=`echo $list | awk -F ":" '{print $1}'`
	PORT=`echo $list | awk -F ":" '{print $2}'`
#	echo $IP	#测试日志打印
#	echo $PORT	#测试日志打印
	scan $IP $PORT
done

#睡1分钟，再次监测，避免误报
sleep 1m

for clist in ${cheaklist[@]}
do
#	echo $clist	  #测试日志打印
	IP=`echo $clist | awk -F ":" '{print $1}'`
	PORT=`echo $clist | awk -F ":" '{print $2}'`
#	echo $IP	#测试日志打印
#	echo $PORT	#测试日志打印
	rescan $IP $PORT
done


```

