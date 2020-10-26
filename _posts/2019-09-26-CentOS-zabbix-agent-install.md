---
layout: post
title: Zabbix-agent centos 部署脚本
categories: Zabbix
description: Zabbix-agent centos 部署脚本
keywords: cengos, zabbix, ageng, 部署脚本
---

# Zabbix-agent centos 部署脚本

- 内网DNS：172.16.252.252
- 内网CentOS7 yum源：yum7.sunimc.cn
- 内网CentOS6 yum源：yum6.sunimc.cn
- Zabbix服务域名：zabbix.sunimc.cn
- edit by Minggle
- [zabbix_install.sh](http://mingsec.com/shell/zabbix_install.sh)

```shell
#!/bin/bash
# zabbix-agent自动化安装脚本
# date:2019.09.24
# edit by Minggle
# 检测DNS
function cheak_dns(){
	echo "正在检测DNS..."
	dns_info=$(cat /etc/resolv.conf | grep 172.16.252.252)
	if [ "$dns_info" ];then
		echo "DNS配置正常。"
		return 0
	else
		echo -e "\033[41;30m DNS配置不正确，请检查DNS配置后重新运行脚本。 \033[0m"
		exit 1
	fi
}

# 检测YUM源
function cheak_yum(){
	echo "正在检测Yum源..."
	yum_info=$(cat /etc/yum.repos.d/CentOS-Base.repo | grep -E yum.\.sunimc\.cn)
	if [ "$yum_info" ];then
		echo "Yum源检测正常。"
		return 0
	else
		echo -e "\033[41;30m Yum源配置不正确，请检测Yum源配置文件后重新运行脚本。 \033[0m"
		exit 1
	fi
}


# 检测网络连通性
function cheak_network(){
	echo "正在检测网络配置..."
	network_info=$(curl -l -s -o /dev/null -w %{http_code}  yum7.sunimc.cn)
	if [ $network_info == 200 ];then
		echo "网络配置检测正常。"
		return 0
	else
		echo -e "\033[41;30m 网络配置检测异常，请检测网络后重新运行脚本。 \033[0m"
		exit 1
	fi
}

# 检测zabbix_agent安装
function zabbix_install(){
	echo "正在监测zabbix-agent。"
	if [ ! "$(rpm -qa | grep zabbix-agent)" ];then
		echo "未检测到zabbix-agent，启动安装程序"
		yum -y install zabbix-agent.x86_64
		if [ "$(rpm -qa | grep zabbix-agent)" ];then
			echo "zabbix-agent 安装完成。"
			return 0
		else
			echo -e "\033[41;30m abbix-agent 安装失败。请检查错误并重新运行脚本。 \033[0m"
			exit 1
		fi
	else
		echo "检测到zabbix-agent客户端。"
		return 0
	fi
}

# zabbix agent配置
function zabbix_config(){
	echo "启动zabbix-agent配置进程。。。"
	cp /etc/zabbix/zabbix_agentd.conf /etc/zabbix/zabbix_agentd.conf.bak
	read -p "请输入系统名称：" Host_Name
	sed 's|127.0.0.1|zabbix.sunimc.cn|g' /etc/zabbix/zabbix_agentd.conf -i
	sed "s|=Zabbix server|=$Host_Name|g" /etc/zabbix/zabbix_agentd.conf -i
	echo "zabbix-agent配置完成。"
}

# zabbix agent 启动与防火墙配置
function zabbix_start(){
	echo "启动Zabbix-agent进程。。。"
	sys_info7=$(cat /etc/redhat-release | awk '{print $4}' | cut -c 1)
	sys_info6=$(cat /etc/redhat-release | awk '{print $3}' | cut -c 1)
	if [ $sys_info7 == 7 ];then
		systemctl enable zabbix-agent
		systemctl start zabbix-agent
		firewall-cmd --zone=public --add-port=10050/tcp --permanent
		firewall-cmd --reload
		return 0
	elif [ $sys_info6 == 6 ];then
		service zabbix-agent start
		chkconfig --level 2345 zabbix-agent on
		iptables -I INPUT -p tcp -m state --state NEW -m tcp --dport 10050 -j ACCEPT
		/etc/init.d/iptables save
		return 0
	else
		echo -e "\033[41;30m 系统监测出错，请重新运行脚本。 \033[0m"
		exit 1
	fi
}


# zabbix agent 状态检测
function zabbix_cheak(){
	echo "启动Zabbix-agent状态检测。。。"
	if [ "$(ps -ef | grep zabbix_agentd | grep -v grep)" ];then
		return 0
		echo "Zabbix启动状态正常。"
	else
		echo -e "\033[41;30m Zabbix状态异常，请检查服务是否正常！！！！！！ \033[0m"
		exit 1
	fi
}

echo "Zabbix-agent安装程序启动，初始化检测中..."
cheak_dns
cheak_yum
cheak_network
zabbix_install
zabbix_config
zabbix_start
zabbix_cheak
echo -e "\033[47;34m Zabbix-agent安装完成，请完成Server端配置。 \033[0m"
```

