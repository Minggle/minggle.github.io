---
layout: post
title: 基础扫描工具
categories: Security
description: 基础扫描工具
keywords: 扫描
---

### 0x01 httrack 
- 侦查工具


- 备份目标文件到本地（减少与目标交互）

- 代理ip

- hidemyass.com(翻墙）

```shell
root@kali-64b:~# httrack 

Welcome to HTTrack Website Copier (Offline Browser) 3.49-2
Copyright (C) 1998-2017 Xavier Roche and other contributors
To see the option list, enter a blank line or try httrack --help

Enter project name :dvwa

Base path (return=/root/websites/) :/root/dvwa  

Enter URLs (separated by commas or blank spaces) :http://172.16.76.234/dvwa/ 

Action:
(enter)	1	Mirror Web Site(s)
	2	Mirror Web Site(s) with Wizard
	3	Just Get Files Indicated
	4	Mirror ALL links in URLs (Multiple Mirror)
	5	Test Links In URLs (Bookmark Test)
	0	Quit
: 2

Proxy (return=none) :

You can define wildcards, like: -*.gif +www.*.com/*.zip -*img_*.zip
Wildcards (return=none) :*

You can define additional options, such as recurse level (-r<number>), separated by blank spaces
To see the option list, type help
Additional options (return=none) :

---> Wizard command line: httrack http://172.16.76.234/dvwa/ -W -O "/root/dvwa/dvwa"  -%v  *

Ready to launch the mirror? (Y/n) :Y

WARNING! You are running this program as root!
It might be a good idea to run as a different user
Mirror launched on Tue, 12 Sep 2017 15:21:32 by HTTrack Website Copier/3.49-2 [XR&CO'2014]
mirroring http://172.16.76.234/dvwa/ * with the wizard help..
```
---
### 0x02 nikto #扫描工具
---
- 更新

```shell
nikto -update          #不成功则可以访问网站下载http://cirt.net/nikto/UPDATES
```
- 扫描命令

```shell
nikto -host http://1.1.1.1      #扫描地址
nikto -host 192.168.1.1 -ssl -port 443      #扫描ssl
nikto -host host.txt        #批量扫描
nmap -p 80 192.168.1.0/24 -oG - | nikto -host -       #NMAP配合扫描
nikto -host 192.168.1.1 -useproxy http://localhost:8087       #代理模式
-vhost              #虚拟端口
vi /etc/nikto.conf              #修改cookie
-evasion			#逃避ids机制

```

---
### 0x03 vega   

---
- 安装

```shell
apt-get install vega
```

- 代理模式

- GoAgent

- CA证书认证

http://vega/ca.crt

---
### 0x04 skipfish
C语言编写，速度较快

```shell
root@kali-64b:~# skipfish -h
skipfish web application scanner - version 2.10b
Usage: skipfish [ options ... ] -W wordlist -o output_dir start_url [ start_url2 ... ]

Authentication and access options:

  -A user:pass      - use specified HTTP authentication credentials
  -F host=IP        - pretend that 'host' resolves to 'IP'
  -C name=val       - append a custom cookie to all requests
  -H name=val       - append a custom HTTP header to all requests
  -b (i|f|p)        - use headers consistent with MSIE / Firefox / iPhone
  -N                - do not accept any new cookies
  --auth-form url   - form authentication URL
  --auth-user user  - form authentication user
  --auth-pass pass  - form authentication password
  --auth-verify-url -  URL for in-session detection

Crawl scope options:

  -d max_depth     - maximum crawl tree depth (16)
  -c max_child     - maximum children to index per node (512)
  -x max_desc      - maximum descendants to index per branch (8192)
  -r r_limit       - max total number of requests to send (100000000)
  -p crawl%        - node and link crawl probability (100%)
  -q hex           - repeat probabilistic scan with given seed
  -I string        - only follow URLs matching 'string'
  -X string        - exclude URLs matching 'string'
  -K string        - do not fuzz parameters named 'string'
  -D domain        - crawl cross-site links to another domain
  -B domain        - trust, but do not crawl, another domain
  -Z               - do not descend into 5xx locations
  -O               - do not submit any forms
  -P               - do not parse HTML, etc, to find new links

Reporting options:

  -o dir          - write output to specified directory (required)
  -M              - log warnings about mixed content / non-SSL passwords
  -E              - log all HTTP/1.0 / HTTP/1.1 caching intent mismatches
  -U              - log all external URLs and e-mails seen
  -Q              - completely suppress duplicate nodes in reports
  -u              - be quiet, disable realtime progress stats
  -v              - enable runtime logging (to stderr)

Dictionary management options:

  -W wordlist     - use a specified read-write wordlist (required)
  -S wordlist     - load a supplemental read-only wordlist
  -L              - do not auto-learn new keywords for the site
  -Y              - do not fuzz extensions in directory brute-force
  -R age          - purge words hit more than 'age' scans ago
  -T name=val     - add new form auto-fill rule
  -G max_guess    - maximum number of keyword guesses to keep (256)

  -z sigfile      - load signatures from this file

Performance settings:

  -g max_conn     - max simultaneous TCP connections, global (40)
  -m host_conn    - max simultaneous connections, per target IP (10)
  -f max_fail     - max number of consecutive HTTP errors (100)
  -t req_tmout    - total request response timeout (20 s)
  -w rw_tmout     - individual network I/O timeout (10 s)
  -i idle_tmout   - timeout on idle HTTP connections (10 s)
  -s s_limit      - response size limit (400000 B)
  -e              - do not keep binary responses for reporting

Other settings:

  -l max_req      - max requests per second (0.000000)
  -k duration     - stop scanning after the given duration h:m:s
  --config file   - load the specified configuration file

Send comments and complaints to <heinenn@google.com>.

```

- 扫描

```shell
skipfish -o test http://1.1.1.1
skipfish -o test @url.txt
-I #只扫描包含‘关键字’的URL
-X #排除包含‘关键字’的URL
-S #制定目录字典
-W #写入特定字符进字典#.wl


```

---
### 0x05 w3af
web application attack and audit framework

9个功能模块介绍
```shell
audi #审计类，扫描发现漏洞操作，eg，注入，xss等
infrastructure  #基础架构，操作系统，webserver，waf
grep    #被动扫描插件类型，根据audi流量进行信息搜索，发现异常信息
evasion    #逃避目标IDS\IPS\WAF检测
mangle    #基于正则表达式的内容替换
auth    #身份认证
bruteforce    #暴力破解
output    #扫描输出
crawl    #爬网
attack
```
- 安装

```shell
root@kali-64b:~# cd ~
root@kali-64b:~# apt-get update 
root@kali-64b:~# apt-get install -y python-pip w3af
root@kali-64b:~# pip install --upgrade pip
root@kali-64b:~# git clone https://github.com/andresriancho/w3af
root@kali-64b:~# cd w3af/
root@kali-64b:~/w3af# ./w3af_console 
root@kali-64b:/tmp# apt-get build-dep python-lxml     #可选
root@kali-64b:~/w3af# cd /tmp
root@kali-64b:/tmp# ./w3af_dependency_install.sh




```

- 升级

```
cd ~/w3af
git pull

```
- 启动

- 快捷方式建立

```shell
/usr/share/application/w3af.desktop
```

---
### 0x06 arachni


下载
```shell
http://www.arachni-scanner.com/download/#linux
```
安装shell
```
tar xvf axxxxxxxx
```
---

