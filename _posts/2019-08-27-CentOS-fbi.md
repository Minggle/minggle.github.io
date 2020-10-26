---
layout: post
title: Centos 系统资源使用情况监测脚本
categories: CentOS
description: Centos 系统资源使用情况监测脚本
keywords: centos, 资源监测
---

- 半小时监测一次服务器系统资源使用情况，如超过设置阈值，则通过短信api接口发送告警短信

- [fbi_warning_2.0.sh](http://mingsec.com/shell/fbi_warning_2.0.sh "fbi_warning_2.0.sh")
- edit by Minggle
- 修改日期：20190815

```shell
#!/bin/bash
source /etc/profile
source /root/.bashrc
#脚本获取CPU、内存、硬盘信息
#conrtab -e
#service crond restart
#0,30 * * * * /bin/bash /usr/local/src/fbi_warning.sh > /dev/null 2>&1
# 获取IP地址
IP_addr=`ip addr | grep inet | grep brd | grep -v vir | awk '{print $2}' | awk -F '/' '{print $1}'`

# 获取当前系统时间
DATE_now=`date -R | awk '{print $2","$3","$4","$5}'`

# 定义阈值
MEM_lim=90
CPU_lim=90
DISK_lim=90

# 定义硬盘排除列表eg.DISK_paichu='(tmpfs|/dev)'
DISK_paichu='(tmpfs|cdrom)'

# 获取硬盘使用率


DISK_1_bili=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==1 {print $5}' | awk -F '%' '{print $1}'`
DISK_1_name=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==1 {print $6}'`
DISK_1_total=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==1 {print $2}'`
DISK_1_used=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==1 {print $3}'`
DISK_1_free=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==1 {print $4}'`

DISK_2_bili=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==2 {print $5}' | awk -F '%' '{print $1}'`
DISK_2_name=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==2 {print $6}'`
DISK_2_total=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==2 {print $2}'`
DISK_2_used=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==2 {print $3}'`
DISK_2_free=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==2 {print $4}'`


DISK_3_bili=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==3 {print $5}' | awk -F '%' '{print $1}'`
DISK_3_name=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==3 {print $6}'`
DISK_3_total=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==3 {print $2}'`
DISK_3_used=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==3 {print $3}'`
DISK_3_free=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==3 {print $4}'`


DISK_4_bili=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==4 {print $5}' | awk -F '%' '{print $1}'`
DISK_4_name=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==4 {print $6}'`
DISK_4_total=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==4 {print $2}'`
DISK_4_used=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==4 {print $3}'`
DISK_4_free=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==4 {print $4}'`


DISK_5_bili=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==5 {print $5}' | awk -F '%' '{print $1}'`
DISK_5_name=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==5 {print $6}'`
DISK_5_total=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==5 {print $2}'`
DISK_5_used=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==5 {print $3}'`
DISK_5_free=`df -hP | grep -v 文件系统 | grep -v Filesystem | grep -vE $DISK_paichu | awk 'NR==5 {print $4}'`



# 获取内存使用率
MEM_total=`free -m | grep Mem |awk -F '[ :]+' '{print $2}'`
MEM_used_all=`free -m | grep Mem |awk -F '[ :]+' '{print $3}'`
#MEM_used_cache=`free -m | grep "buffers/cache" |awk -F '[ :]+' '{print $3}'`
MEM_used=`free -m | grep Mem |awk -F '[ :]+' '{print $3-$6-$7}'`
#MEM_used=$[ MEM_used_all-MEM_used_cache ]
MEM_free=$[ MEM_total-MEM_used ]
#MEM_free=`free -m | awk -F '[ :]+' 'NR==2{print $4}'`
MEM_bili_0=$[ MEM_used*100/MEM_total]
MEM_bili=$[ MEM_used*100/MEM_total]
#MEM_bili_0=`free -m | awk -F '[ :]+' 'NR==2{print $3*100/$2}'`
#MEM_bili=`free -m | awk -F '[ :]+' 'NR==2{print $3*100/$2}' | awk -F '.' '{print $1}' `


# 判断输出
# 内存使用率是否超过阈值
if [ $MEM_bili -ge $MEM_lim ]; then
	NOTE_mem="告警：$DATE_now，目前$IP_addr系统内存使用率"$MEM_bili_0"%，系统总内存"$MEM_total"M，空闲内存"$MEM_free"M，已使用内存"$MEM_used"M,已超过设定阈值，请排查进程内存使用情况是否正常。"
	curl http://xxxx/notice  -X POST -H "Content-Type:application/json" -d '{"ip":"'$IP_addr'","content":"'$NOTE_mem'"}'
	echo $NOTE_mem
	echo -e "告警：$DATE_now , 目前 $IP_addr 系统内存使用率 $MEM_bili_0 %，系统总内存 $MEM_total M, 空闲内存 $MEM_free M, 已使用内存 $MEM_used M。" >> /usr/local/src/memlog.log
else
	echo -e "日志：$DATE_now , 目前 $IP_addr 系统内存使用率 $MEM_bili_0 %，系统总内存 $MEM_total M, 空闲内存 $MEM_free M, 已使用内存 $MEM_used M。" >> /usr/local/src/memlog.log
fi



# 获取CPU使用率
CPU_stat5=`vmstat 1 5|awk '{x+=$13;y+=$14}END{print x+y}'`
CPU_7_0=`echo "scale=2;$CPU_stat5/5"|bc`
CPU_7=`echo $CPU_7_0 | awk -F '.' '{print $1}'`

# 判断输出
# CPU使用率是否超过阈值
if [ ! $CPU_7 ];then
	echo -e "日志：$DATE_now , 目前 $IP_addr 系统CPU使用率 0"$CPU_7_0" %。" >> /usr/local/src/cpulog.log
else
	if [ $CPU_7 -ge $CPU_lim ]; then
		NOTE_cpu="告警：$DATE_now，目前$IP_addr系统CPU使用率$CPU_7_0%，已超过设定阈值，请排查进程CPU使用情况是否正常。"
		echo $NOTE_cpu
		curl http://192.168.91.4:9999/Sunriver/notice  -X POST -H "Content-Type:application/json" -d '{"ip":"'$IP_addr'","content":"'$NOTE_cpu'"}'
		echo -e "告警：$DATE_now , 目前 $IP_addr 系统CPU使用率 $CPU_7_0 %。" >> /usr/local/src/cpulog.log
	else
		echo -e "日志：$DATE_now , 目前 $IP_addr 系统CPU使用率 $CPU_7_0 %。" >> /usr/local/src/cpulog.log
	
	fi
fi

# 硬盘使用率是否超过阈值

if [ $DISK_1_bili ]; then
	if [ $DISK_1_bili -ge $DISK_lim ];then
		NOTE_disk="告警：$DATE_now，目前$IP_addr系统$DISK_1_name目录硬盘使用率为$DISK_1_bili%，目录硬盘总空间"$DISK_1_total"，空闲空间"$DISK_1_free",已使用空间"$DISK_1_used"，已超过设定阈值，请排查硬盘使用情况是否正常。"
		echo $NOTE_disk
		curl http://xxxx/notice  -X POST -H "Content-Type:application/json" -d '{"ip":"'$IP_addr'","content":"'$NOTE_disk'"}'
		echo -e "告警：$DATE_now，目前$IP_addr系统$DISK_1_name目录硬盘使用率为$DISK_1_bili。" >>/usr/local/src/disklog.log
	else
		echo -e "日志：$DATE_now，目前$IP_addr系统$DISK_1_name目录硬盘使用率为$DISK_1_bili。" >>/usr/local/src/disklog.log
	fi
fi

if [ $DISK_2_bili ]; then
	if [ $DISK_2_bili -ge $DISK_lim ];then
		NOTE_disk="告警：$DATE_now，目前$IP_addr系统$DISK_2_name目录硬盘使用率为$DISK_2_bili%，目录硬盘总空间"$DISK_2_total"，空闲空间"$DISK_2_free",已使用空间"$DISK_2_used"，已超过设定阈值，请排查硬盘使用情况是否正常。"
		echo $NOTE_disk
		curl http://xxxx/notice  -X POST -H "Content-Type:application/json" -d '{"ip":"'$IP_addr'","content":"'$NOTE_disk'"}'
		echo -e "告警：$DATE_now，目前$IP_addr系统$DISK_2_name目录硬盘使用率为$DISK_2_bili。" >>/usr/local/src/disklog.log
	else
		echo -e "日志：$DATE_now，目前$IP_addr系统$DISK_2_name目录硬盘使用率为$DISK_2_bili。" >>/usr/local/src/disklog.log
	fi
fi

if [ $DISK_3_bili ]; then
	if [ $DISK_3_bili -ge $DISK_lim ];then
		NOTE_disk="告警：$DATE_now，目前$IP_addr系统$DISK_3_name目录硬盘使用率为$DISK_3_bili%，目录硬盘总空间"$DISK_3_total"，空闲空间"$DISK_3_free",已使用空间"$DISK_3_used"，已超过设定阈值，请排查硬盘使用情况是否正常。"
		echo $NOTE_disk
		curl http://xxxx/notice  -X POST -H "Content-Type:application/json" -d '{"ip":"'$IP_addr'","content":"'$NOTE_disk'"}'
		echo -e "告警：$DATE_now，目前$IP_addr系统$DISK_3_name目录硬盘使用率为$DISK_3_bili。" >>/usr/local/src/disklog.log
	else
		echo -e "日志：$DATE_now，目前$IP_addr系统$DISK_3_name目录硬盘使用率为$DISK_3_bili。" >>/usr/local/src/disklog.log
	fi
fi

if [ $DISK_4_bili ]; then
	if [ $DISK_4_bili -ge $DISK_lim ];then
		NOTE_disk="告警：$DATE_now，目前$IP_addr系统$DISK_4_name目录硬盘使用率为$DISK_4_bili%，目录硬盘总空间"$DISK_4_total"，空闲空间"$DISK_4_free",已使用空间"$DISK_4_used"，已超过设定阈值，请排查硬盘使用情况是否正常。"
		echo $NOTE_disk
		curl http://xxxx/notice  -X POST -H "Content-Type:application/json" -d '{"ip":"'$IP_addr'","content":"'$NOTE_disk'"}'
		echo -e "告警：$DATE_now，目前$IP_addr系统$DISK_4_name目录硬盘使用率为$DISK_4_bili。" >>/usr/local/src/disklog.log
	else
		echo -e "日志：$DATE_now，目前$IP_addr系统$DISK_4_name目录硬盘使用率为$DISK_4_bili。" >>/usr/local/src/disklog.log
	fi
fi

if [ $DISK_5_bili ]; then
	if [ $DISK_5_bili -ge $DISK_lim ];then
		NOTE_disk="告警：$DATE_now，目前$IP_addr系统$DISK_5_name目录硬盘使用率为$DISK_5_bili%，目录硬盘总空间"$DISK_5_total"，空闲空间"$DISK_5_free",已使用空间"$DISK_5_used"，已超过设定阈值，请排查硬盘使用情况是否正常。"
		echo $NOTE_disk
		curl http://xxxx/notice  -X POST -H "Content-Type:application/json" -d '{"ip":"'$IP_addr'","content":"'$NOTE_disk'"}'
		echo -e "告警：$DATE_now，目前$IP_addr系统$DISK_5_name目录硬盘使用率为$DISK_5_bili。" >>/usr/local/src/disklog.log
	else
		echo -e "日志：$DATE_now，目前$IP_addr系统$DISK_5_name目录硬盘使用率为$DISK_5_bili。" >>/usr/local/src/disklog.log
	fi
fi
```

