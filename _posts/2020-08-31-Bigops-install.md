---
layout: post
title: BigOps安装过程实录
categories: Bigops
description: BigOps安装过程实录
keywords: bigops, install
---

## 1. 操作系统信息

``` shell
[root@bigops ~]# cat /etc/redhat-release 
CentOS Linux release 7.5.1804 (Core) 
 [root@bigops ~]# grep 'core id' /proc/cpuinfo | sort -u | wc -l
4
[root@bigops ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           7805         961        4755          15        2087        6474
Swap:          8063           0        8063

```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200828172311371.png)

### 1.1 设置DNS

```shell
[root@bigops ~]# echo "192.168.16.131 sso.bigops.com" >> /etc/hosts
[root@bigops ~]# echo "192.168.16.131 work.bigops.com" >> /etc/hosts
```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200828172425636.png)

### 1.2 初始化环境

```shell
[root@bigops ~]# cd ~
[root@bigops ~]# yum -y groupinstall "Development Tools"
[root@bigops ~]# yum -y install wget
[root@bigops ~]# wget -O centos_init_env.sh http://www.bigops.com/bigops-install/centos_init_env.sh
[root@bigops ~]# bash centos_init_env.sh
```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200828172558091.png)

### 1.3 关闭IPv6

```shell
[root@bigops ~]# cp /etc/sysctl.conf /etc/sysctl.conf.back
[root@bigops ~]# echo "net.ipv6.conf.all.disable_ipv6 = 1" >> /etc/sysctl.conf
[root@bigops ~]# echo "net.ipv6.conf.default.disable_ipv6 = 1" >>/etc/sysctl.conf
```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200828172701651.png)

```shell
[root@bigops ~]# cp /etc/sysconfig/network /etc/sysconfig/network.backup
[root@bigops ~]# echo "NETWORKING_IPV6=no" >>/etc/sysconfig/network
```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200828172756046.png)

```shell
[root@bigops ~]# sed s/IPV6/#IPV6/g /etc/sysconfig/network-scripts/ifcfg-e* -i
[root@bigops ~]# echo "IPV6INIT=no" >> /etc/sysconfig/network-scripts/ifcfg-e*
```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200828172802245.png)

```shell
[root@bigops ~]# systemctl disable NetworkManager
```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200828172808302.png)

## 2. 安装Mysql

### 2.1 脚本安装Mysql 8.0

```shell
[root@bigops ~]# wget -O mysql80.sh http://www.bigops.com/bigops-install/mysql80.sh
[root@bigops ~]# bash mysql80.sh
```

mysql80.sh脚本如下：

```shell
#!/bin/bash

yum -y install net-tools numactl pkgconfig perl perl-DBI perl-Compress-Raw-Bzip2 perl-Net-Daemon
yum -y install perl-Module-Pluggable perl-Pod-Escapes perl-Pod-Simple perl-libs perl-version
yum -y install perl-parent perl-Pod-Escapes perl-Pod-Simple perl-Time-HiRes perl-libs openssl-devel openssl
yum -y install libaio libaio-devel

if [ -f /usr/bin/systemctl ];then
    systemctl stop mysqld.service
else
    service mysqld stop
fi

if [ ! -z "$(ps aux|grep mysqld|grep -v grep)" ];then 
   ps aux|grep mysqld|grep -v grep|awk '{print $2}'|xargs kill -9
fi

if [ ! -d /opt/mysql-rpms ];then
   mkdir -p /opt/mysql-rpms
fi

inst(){
    if [ ! -f /usr/sbin/mysqld ];then
        echo "not found /usr/sbin/mysqld, installation failed"
        exit
    fi
    wget -O /etc/my.cnf http://www.bigops.com/bigops-install/mysql/my-80.cnf
    if [ -f /usr/lib/systemd/system/mysqld.service ];then
        sed -i 's/LimitNOFILE = 10000/LimitNOFILE = 65535/g' /usr/lib/systemd/system/mysqld.service
        systemctl daemon-reload
    fi
    chmod 644 /etc/my.cnf

    echo -e "Confirm delete database datadir: /var/lib/mysql/ , (y/n)?"
    echo -e ">\c"
    read sure

    if [ "$sure" == 'y' ];then
        rm -rf /var/lib/mysql/*     
    else
        echo "exit install"
        exit
    fi

    if [ ! -e /var/lib/mysql-files ];then
        mkdir /var/lib/mysql-files
    fi
    chmod 777 /var/lib/mysql-files
    chown -R mysql:mysql /var/lib/mysql

    mysqld --user=mysql --lower-case-table-names=0 --initialize-insecure
    chown -R mysql:mysql /var/lib/mysql
    
    if [ -f /usr/bin/systemctl ];then
        systemctl start mysqld.service
    else
        service mysqld start
    fi

    echo
    echo ----------------------------------
    echo "press any key to continue"
    read

    echo ----------------------------------
    echo -e "please input root@% password, default bigops"
    echo -e ">\c"
    read mypass

    if  [ -z "${mypass}" ];then
        mypass='bigops'
    fi

mysql --connect-timeout 5 -uroot -e "
use mysql;
create user 'root'@'%' identified with mysql_native_password by '${mypass}';
grant all privileges on *.* to 'root'@'%';
flush privileges;
"

    if [ $? == 0 ];then
        echo
        echo ----------------------------------
        echo "Installed successfully, root@% password is ${mypass}"
        echo "please running command testing: mysql -uroot -h127.0.0.1 -p${mypass}"
        echo ----------------------------------
    else
        echo "Installed failure!"
    fi

}

osver=`rpm -qi centos-release|grep Version|awk '{print $NF}'|awk -F\. '{print $1}'`
cd /opt/mysql-rpms/
if [[ "${osver}" == "6" ]] && [[ `arch` == "x86_64" ]];then
    wget -N -c http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-client-8.0.20-1.el6.x86_64.rpm &
    wget -N -c http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-common-8.0.20-1.el6.x86_64.rpm &
    wget -N -c http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-devel-8.0.20-1.el6.x86_64.rpm &
    wget -N -c http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-libs-8.0.20-1.el6.x86_64.rpm &
    wget -N -c http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-libs-compat-8.0.20-1.el6.x86_64.rpm &
    wget -N -c http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-server-8.0.20-1.el6.x86_64.rpm
    rpm -Uvh --force /opt/mysql-rpms/*-8.0*.el6.*.rpm   
    inst
elif [[ "${osver}" == "7" ]] && [[ `arch` == "x86_64" ]];then
    wget -N -c http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-client-8.0.20-1.el7.x86_64.rpm &
    wget -N -c http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-common-8.0.20-1.el7.x86_64.rpm &
    wget -N -c http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-devel-8.0.20-1.el7.x86_64.rpm &
    wget -N -c http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-libs-8.0.20-1.el7.x86_64.rpm &
    wget -N -c http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-libs-compat-8.0.20-1.el7.x86_64.rpm &
    wget -N -c http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-server-8.0.20-1.el7.x86_64.rpm
    rpm -Uvh --force /opt/mysql-rpms/*-8.0*.el7.*.rpm
    inst
elif [[ "${osver}" == "8" ]] && [[ `arch` == "x86_64" ]];then
    wget -N -c http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-client-8.0.20-1.el8.x86_64.rpm &
    wget -N -c http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-common-8.0.20-1.el8.x86_64.rpm &
    wget -N -c http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-devel-8.0.20-1.el8.x86_64.rpm &
    wget -N -c http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-libs-8.0.20-1.el8.x86_64.rpm &
    wget -N -c http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-libs-compat-8.0.20-1.el8.x86_64.rpm &
    wget -N -c http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-server-8.0.20-1.el8.x86_64.rpm
    rpm -Uvh --force /opt/mysql-rpms/*-8.0*.el8.*.rpm
    inst
else
    echo "不支持当前操作系统：$(rpm -qi centos-release|grep Version|awk '{print $NF}')"
fi



```

### 2.2 设置本地登录权限

```shell
[root@bigops ~]# mysql –uroot –h127.0.0.1 –p
mysql> alter user 'root'@'localhost' identified with mysql_native_password by 'bigops';
mysql> flush privileges;
```

### 2.3 新建用户

```shell
[root@bigops ~]# mysql –uroot –p
mysql> use mysql;
mysql> create user 'bigops' @'%' identified by 'bigops';
mysql> grant all on *.* to 'bigops'@'%';
mysql> flush privileges;	
```

### 2.4 重启服务

```shell
[root@bigops ~]# systemctl restart mysqld
[root@bigops ~]# systemctl status mysqld –l
```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830191208290.png)

## 3. 安装ELK

### 3.1 脚本安装ELK

```shell
[root@bigops ~]# cd ~
[root@bigops ~]# wget -O elk762.sh http://www.bigops.com/bigops-install/elk762.sh
[root@bigops ~]# bash elk762.sh
```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830191455026.png)

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830191502003.png)

elk762.sh脚本如下：

```shell
#!/bin/bash

export PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
export JAVA_HOME=/opt/es/jdk
export PATH=$JAVA_HOME/bin:$PATH

alias mv='mv'
alias rm='rm'

if [ ! -e /usr/bin/systemctl ];then
    echo "不支持当前操作系统，只支持CentOS 7和CentOS 8"
    exit
fi

osver=`rpm -qi centos-release|grep Version|awk '{print $NF}'|awk -F\. '{print $1}'`
if [ "${osver}" == "6" ];then
    echo "不支持当前操作系统，只支持CentOS 7和CentOS 8"
    exit
fi

if [ "$(free -g|awk '/^Mem/{print $2}')" -le 4 ];then
   echo "总内存小于4g，退出！"
   exit
fi

#优化系统
if [ -z "$(grep vm.max_map_count /etc/sysctl.conf)" ];then
   echo 'vm.max_map_count=655360' >>/etc/sysctl.conf
else
   sed -i 's/vm.max_map_count.*/vm.max_map_count=655360/g' /etc/sysctl.conf
fi

if [ -z "$(grep vm.swappiness /etc/sysctl.conf)" ];then
   echo 'vm.swappiness=10' >>/etc/sysctl.conf
else
   sed -i 's/vm.swappiness.*/vm.swappiness=10/g' /etc/sysctl.conf
fi

sysctl -p
ulimit -SHn 655360

#安装依赖软件
#yum -y install java-11-openjdk nodejs npm git bzip2 log4j* net-tools

systemctl stop es 2>/dev/null
systemctl stop kibana 2>/dev/null
systemctl stop logstash 2>/dev/null

#下载软件
if [ ! -e elasticsearch-7.6.2-linux-x86_64.tar.gz ];then
    wget -c https://mirrors.huaweicloud.com/elasticsearch/7.6.2/elasticsearch-7.6.2-linux-x86_64.tar.gz
fi

if [ ! -e elasticsearch-7.6.2-linux-x86_64.tar.gz ];then
   echo "下载 elasticsearch-7.6.2-linux-x86_64.tar.gz 文件失败，退出安装！"
   exit
fi

if [ ! -e kibana-7.6.2-linux-x86_64.tar.gz ];then
    wget -c https://mirrors.huaweicloud.com/kibana/7.6.2/kibana-7.6.2-linux-x86_64.tar.gz
fi

if [ ! -e kibana-7.6.2-linux-x86_64.tar.gz ];then
   echo "下载 kibana-7.6.2-linux-x86_64.tar.gz 文件失败，退出安装！"
   exit
fi

if [ ! -e logstash-7.6.2.tar.gz ];then
    wget -c https://mirrors.huaweicloud.com/logstash/7.6.2/logstash-7.6.2.tar.gz
fi

if [ ! -e logstash-7.6.2.tar.gz ];then
   echo "下载 logstash-7.6.2.tar.gz 文件失败，退出安装！"
   exit
fi

rm -rf elasticsearch-7.6.2 2>/dev/null
rm -rf kibana-7.6.2-linux-x86_64 2>/dev/null
rm -rf logstash-7.6.2 2>/dev/null
rm -rf /opt/es 2>/dev/null
rm -rf /opt/logstash 2>/dev/null
rm -rf /opt/kibana 2>/dev/null

if [ -e /opt/es_data ];then
    echo "/opt/es_data目录已存在，请确认没有数据，删除后在重新进行安装！" 
    exit
fi


#安装es
tar zxvf elasticsearch-7.6.2-linux-x86_64.tar.gz
if [ $? != 0 ];then
    echo "解压错误，文件不完整！"
    exit
fi

cp -rf elasticsearch-7.6.2 /opt/es
useradd es -d /opt/es
mkdir -p /opt/es_data /opt/es_logs
chown -R es:es /opt/es_data /opt/es_logs /opt/es
chmod -R 777 /opt/es_data /opt/es_logs
wget -O /opt/es/config/elasticsearch.yml http://www.bigops.com/bigops-install/elk/elasticsearch.yml

#安装kibana
tar zxvf kibana-7.6.2-linux-x86_64.tar.gz
if [ $? != 0 ];then
    echo "解压错误，文件不完整！"
    exit
fi

cp -rf kibana-7.6.2-linux-x86_64 /opt/kibana
wget -O /opt/kibana/config/kibana.yml http://www.bigops.com/bigops-install/elk/kibana.yml

#安装logstash
tar zxvf logstash-7.6.2.tar.gz
if [ $? != 0 ];then
    echo "解压错误，文件不完整！"
    exit
fi

cp -rf logstash-7.6.2 /opt/logstash

if [ ! -d /opt/logstash-conf ];then
    mkdir /opt/logstash-conf
fi

wget -O /opt/logstash-conf/syslog.conf http://www.bigops.com/bigops-install/elk/syslog.conf
wget -O /opt/logstash-conf/winlog.conf http://www.bigops.com/bigops-install/elk/winlog.conf

esmemsize=2g

sed -i 's/^-Xms1g/-Xms'"${esmemsize}"'/g' /opt/es/config/jvm.options
sed -i 's/^-Xmx1g/-Xmx'"${esmemsize}"'/g' /opt/es/config/jvm.options
sed -i 's/^-XX:+UseConcMarkSweepGC/#-XX:+UseConcMarkSweepGC/g' /opt/es/config/jvm.options
sed -i 's/^-XX:CMSInitiatingOccupancyFraction=75/#-XX:CMSInitiatingOccupancyFraction=75/g' /opt/es/config/jvm.options
sed -i 's/^-XX:+UseCMSInitiatingOccupancyOnly/#-XX:+UseCMSInitiatingOccupancyOnly/g' /opt/es/config/jvm.options

sed -i 's/^8-13:-XX:+UseConcMarkSweepGC/#8-13:-XX:+UseConcMarkSweepGC/g' /opt/es/config/jvm.options
sed -i 's/^8-13:-XX:CMSInitiatingOccupancyFraction=75/#8-13:-XX:CMSInitiatingOccupancyFraction=75/g' /opt/es/config/jvm.options
sed -i 's/^8-13:-XX:+UseCMSInitiatingOccupancyOnly/#8-13:-XX:+UseCMSInitiatingOccupancyOnly/g' /opt/es/config/jvm.options

sed -i '/^-XX:+UseG1GC/d' /opt/es/config/jvm.options
sed -i '/^-XX:MaxGCPauseMillis=.*/d' /opt/es/config/jvm.options
echo '-XX:+UseG1GC' >>/opt/es/config/jvm.options
echo '-XX:MaxGCPauseMillis=200' >>/opt/es/config/jvm.options

echo
echo
echo -e "本机所有IP供参考，如果是云主机弹性IP这里不会显示。"
ifconfig -a|grep inet|awk '{print $2}'|sed 's/addr://g'|grep -Ev '(127.0.0.1|::)'

echo
echo -e "输入当前主机IP，用于ES监听：\c"
read ip

if [ -z "${ip}" ];then
   echo "ES监听IP不能为空，请重新运行安装程序，退出安装！"
   exit
fi

sed -i 's/^node.name:.*/node.name: '"${ip}"'/g' /opt/es/config/elasticsearch.yml
sed -i 's/^cluster.initial_master_nodes:.*/cluster.initial_master_nodes: [\"'"${ip}"':9300\"]/g' /opt/es/config/elasticsearch.yml
sed -i 's/^discovery.seed_hosts:.*/discovery.seed_hosts: [\"'"${ip}"':9300\"]/g' /opt/es/config/elasticsearch.yml

#修改kibana配置文件
echo
echo -e "设置ES连接密码：\c"
read es_pass
if [ ! -z "${es_pass}" ];then
    sed -i "s/^elasticsearch.password:.*/elasticsearch.password: \""${es_pass}"\"/g" /opt/kibana/config/kibana.yml
fi

#修改logstash配置文件
if [ ! -z "${es_pass}" ];then
    sed -i "s#^[ ]*hosts => \[\".*#        hosts => [\""${ip}":9200\"]#g" /opt/logstash-conf/syslog.conf
    sed -i "s#^[ ]*password => \".*#        password => \""${es_pass}"\"#g" /opt/logstash-conf/syslog.conf
    sed -i "s#^[ ]*hosts => \[\".*#        hosts => [\""${ip}":9200\"]#g" /opt/logstash-conf/winlog.conf
    sed -i "s#^[ ]*password => \".*#        password => \""${es_pass}"\"#g" /opt/logstash-conf/winlog.conf
fi

sed -i '/^*.notice @@'"${ip}".*'/d' /etc/rsyslog.conf
echo "*.notice @@${ip}:6514" >/etc/rsyslog.d/bigops.conf
systemctl restart rsyslog

sed -i 's#^LS_HOME=.*#LS_HOME=/opt/logstash#g' /opt/logstash/config/startup.options
sed -i 's#^LS_SETTINGS_DIR=.*#LS_SETTINGS_DIR=/opt/logstash/config#g' /opt/logstash/config/startup.options
sed -i 's#^LS_OPTS=.*#LS_OPTS="--path.settings ${LS_SETTINGS_DIR} -f /opt/logstash-conf"#g' /opt/logstash/config/startup.options
sed -i 's#^LS_USER=.*#LS_USER=root#g' /opt/logstash/config/startup.options
sed -i 's#^LS_GROUP=.*#LS_GROUP=root#g' /opt/logstash/config/startup.options

javahome=/opt/es/jdk
sed -i '/^export PATH=.*/d' /opt/logstash/bin/logstash.lib.sh
sed -i '/^export JAVA_HOME=.*/d' /opt/logstash/bin/logstash.lib.sh
sed -i 'N;2 a export PATH=$JAVA_HOME/bin:$PATH' /opt/logstash/bin/logstash.lib.sh
sed -i "N;2 a export JAVA_HOME="${javahome}"" /opt/logstash/bin/logstash.lib.sh

/opt/logstash/bin/system-install /opt/logstash/config/startup.options systemd

if [ -e /usr/bin/systemctl ];then  
    wget -O /usr/lib/systemd/system/es.service http://www.bigops.com/bigops-install/elk/es.service
    systemctl enable es
    systemctl daemon-reload
    systemctl restart es.service
    systemctl status es.service
    sleep 10
    wget -O /usr/lib/systemd/system/kibana.service http://www.bigops.com/bigops-install/elk/kibana.service
    systemctl enable kibana
    systemctl daemon-reload
    systemctl enable logstash
    systemctl daemon-reload
fi

javahome=/opt/es/jdk
sed -i '/^export PATH=.*/d' /opt/es/bin/elasticsearch
sed -i '/^export JAVA_HOME=.*/d' /opt/es/bin/elasticsearch
sed -i 'N;2 a export PATH=$JAVA_HOME/bin:$PATH' /opt/es/bin/elasticsearch
sed -i "N;2 a export JAVA_HOME="${javahome}"" /opt/es/bin/elasticsearch
sed -i '/^export PATH=.*/d' /opt/es/bin/elasticsearch-env
sed -i '/^export JAVA_HOME=.*/d' /opt/es/bin/elasticsearch-env
sed -i 'N;2 a export PATH=$JAVA_HOME/bin:$PATH' /opt/es/bin/elasticsearch-env
sed -i "N;2 a export JAVA_HOME="${javahome}"" /opt/es/bin/elasticsearch-env
sed -i '/^export PATH=.*/d' /opt/es/bin/elasticsearch-setup-passwords
sed -i '/^export JAVA_HOME=.*/d' /opt/es/bin/elasticsearch-setup-passwords
sed -i 'N;2 a export PATH=$JAVA_HOME/bin:$PATH' /opt/es/bin/elasticsearch-setup-passwords
sed -i "N;2 a export JAVA_HOME="${javahome}"" /opt/es/bin/elasticsearch-setup-passwords

echo
echo "后续步骤一：设置ES密码"
echo "-------------------"
echo "等待30秒后，查看ES是否启动，运行命令："
echo "netstat -nptl|grep 9[2,3]00"
echo "如果没有启动，请尝试手动启动进行排错："
echo "su - es"
echo "/opt/es/bin/elasticsearch"
echo 
echo "9200和9300端口启动后，运行下面命令设置密码"
echo "/opt/es/bin/elasticsearch-setup-passwords interactive"
echo "Please confirm that you would like to continue [y/N]，回答y"
echo "要设置的密码比较多，都设置成ES连接密码"
echo "-------------------"
echo 
echo "后续步骤二：启动kibana"
echo "-------------------"
echo "运行命令"
echo "systemctl restart kibana.service"
echo
echo "等待30秒后，查看端口是否启动，运行命令"
echo "netstat -nplt|grep 5601"
echo
echo "5601端口启动后，使用浏览器访问：http://${ip}:5601"
echo "默认登录用户名：elastic"
echo "密码：刚才设置的ES连接密码"
echo
echo "后续步骤三：启动logstash"
echo "-------------------"
echo "运行命令"
echo "systemctl restart logstash.service"
echo
echo "等待30秒后，查看端口是否启动，运行命令"
echo "netstat -npl|grep 6514"
echo "-------------------"
echo

```


### 3.2 查看ES是否启动

```shell
[root@bigops ~]# netstat -nptl|grep 9[2,3]00
```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830191522530.png)

### 3.3 修改密码

```shell
[root@bigops ~]# /opt/es/bin/elasticsearch-setup-passwords interactive
```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830191541587.png)

### 3.4 启动kibana

```shell
[root@bigops ~]# systemctl restart kibana.service
[root@bigops ~]# netstat -nplt|grep 5601
```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830191603529.png)

- 登录网页查看是否正常
  - <-http://192.168.16.131:5601>
  - `username`:`elastic`
  - `password`:`bigops`

### 3.5 启动logstash

```shell
[root@bigops ~]# systemctl restart logstash.service
[root@bigops ~]# netstat -npl|grep 6514
```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830191816412.png)

### 3.6 启动cerebro

```shell
[root@bigops ~]# cd ~
[root@bigops ~]# wget -O cerebro.sh http://www.bigops.com/bigops-install/cerebro.sh
[root@bigops ~]# bash cerebro.sh
```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830191835781.png)

cerebro.sh脚本如下
```shell
#!/bin/bash

export PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

alias mv='mv'
alias rm='rm'

if [ ! -z "$(ps aux|grep cerebro.cerebro|grep -v grep)" ];then
   ps aux|grep cerebro.cerebro|grep -v grep|awk '{print $2}'|xargs kill -9
fi


#下载软件
cd ~
wget -c https://github.com/lmenezes/cerebro/releases/download/v0.8.5/cerebro-0.8.5.tgz

if [ -e cerebro-0.8.5 ];then
   rm -rf cerebro-0.8.5
fi

tar zxvf cerebro-0.8.5.tgz

if [ -e /opt/cerebro ];then
    echo "/opt/cerebro目录已存在，请确认没有数据，删除后在重新进行安装！" 
    exit
fi

mv cerebro-0.8.5 /opt/cerebro

mv /opt/cerebro/conf/application.conf /opt/cerebro/conf/application.confbak
wget -O /opt/cerebro/conf/application.conf http://www.bigops.com/bigops-install/elk/application.conf

esip=`awk '/cluster.initial_master_nodes/{print $NF}' /opt/es/config/elasticsearch.yml|awk -F: '{print $1}'|sed 's/\[\"//g'`
sed -i 's#^[ ]*host =.*#    host = \"http://'"${esip}"':9200"#g' /opt/cerebro/conf/application.conf

cluster_name=`awk '/cluster.name/{print $NF}' /opt/es/config/elasticsearch.yml|sed 's/[ ]*//g'`
sed -i 's#^[ ]*name =.*#    name = \"'"${cluster_name}"'"#g' /opt/cerebro/conf/application.conf

wget -O /usr/lib/systemd/system/cerebro.service http://www.bigops.com/bigops-install/elk/cerebro.service
chmod -R 777 /opt/cerebro/bin/
systemctl enable cerebro
systemctl daemon-reload
systemctl restart cerebro.service

echo
echo "后续步骤一："
echo "-------------------"
echo "等待5秒后，查看9000端口是否启动。运行命令："
echo "netstat -nptl|grep 9000"
echo "-------------------"
echo 
echo "后续步骤二："
echo "-------------------"
echo "9000端口启动后，使用浏览器访问：http://${esip}:9000，会显示一个登陆页面"
echo "Node address：输入http://localhost:9200，然后点Connect按钮"
echo "Username：输入ES连接用户"
echo "Password：输入ES连接密码"
echo "-------------------"
echo


```

### 3.7 查看9000端口是否启动

```shell
[root@bigops ~]# netstat -nptl|grep 9000
```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830191855162.png)

- 登录网页查看是否正常
  - <http://192.168.16.131:9000>
  - `user`name:`elastic`
  - `password`:`bigops`

## 4. 安装升级BigOps

### 4.1 上传安装包

上传安装包到/opt目录

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830192404728.png)

### 4.2 解压缩

```shell
[root@bigops opt]# tar zxf bigops-3.0.7-install.tar.gz
```

### 4.3 运行安装脚本

```shell
[root@bigops opt]# cd /opt/bigops-3.0.7-install/
[root@bigops bigops-3.0.7-install]# bash install.sh
```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830192449622.png)

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830191455026.png)

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830191455026.png)



![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830191455026.png)

### 4.4 确认安装状态

#### 4.4.1 检查SSO及WORK服务是否正常

```shell
 [root@bigops bigops-3.0.7-install]# curl -q 127.0.0.1:30001/signin/login 2>/dev/null |grep sso
<h1>sso系统正常!!</h1>
[root@bigops bigops-3.0.7-install]# curl 127.0.0.1:30003/api/common/ssourl/
{"code":0,"message":"ok","data":{"url":"http://sso.bigops.com"}}

```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830191455026.png)

#### 4.4.2 检查系统运行状态

```shell
[root@bigops bigops-3.0.7-install]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS                                                NAMES
8dfb2b4375c8        bigops:latest       "/root/start.sh"    About a minute ago   Up About a minute   0.0.0.0:30001->30001/tcp, 0.0.0.0:30003->30003/tcp   bigops
```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830191455026.png)

#### 4.4.3 检查服务端口

```shell
[root@bigops bigops-3.0.7-install]# netstat -nptl|grep docker
```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830191455026.png)

#### 4.4.4 检查Nginx域名是否配置正确

```shell
[root@bigops bigops-3.0.7-install]# cat /etc/nginx/conf.d/sso.conf
```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830193121191.png)

```shell
[root@bigops bigops-3.0.7-install]# cat /etc/nginx/conf.d/work.conf
```

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830193121191.png)

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830193121191.png)

### 4.5 登录检查

- 本地客户端写入`host`

```powershell
192.168.16.131	work.bigops.com
192.168.16.131	sso.bigops.com
```

- <http://work.bigops.com>
  - `username`:`admin`
  - `password`:`bigops`

### 4.6 设置ES连接

点击桌面/日志/ES设置

![](https://raw.githubusercontent.com/Minggle/image/main/image/image-20200830193121191.png)

## 5. 安装Proxy

