---
layout: post
title: BurpSuite安装
categories: BurpSuite
description: BurpSuite安装
keywords: burpsuite
---

# 安装

### 安装java
- 下载java

```shell
http://www.oracle.com/technetwork/java/javase/downloads/index.html
```
```shell
tar zxvf jdk-8u144-linux-x64.tar.gz
mv jdk1.8.0_144/ /opt


update-alternatives --install /usr/bin/java java /opt/jdk1.8.0_144/bin/java 1
update-alternatives --install /usr/bin/javac javac /opt/jdk1.8.0_144/bin/javac 1
update-alternatives --install /usr/lib/mozilla/plugins/libjavaplugin.so mozilla-javaplugin.so /opt/jdk1.8.0_144/jre/lib/amd64/libnpjp2.so 1
update-alternatives --set java /opt/jdk1.8.0_144/bin/java
update-alternatives --set javac /opt/jdk1.8.0_144/bin/javac
update-alternatives --set mozilla-javaplugin.so /opt/jdk1.8.0_144/jre/lib/amd64/libnpjp2.so

java -version
```


- 下载使用burpsuite

```shell
mkdir burp
vi burp.sh
    java -cp BurpLoader.jar larry.lau.BurpLoader
chmod +x burp.sh
./burp.sh



```
```shell
#linux.sh运行脚本
/opt/jdk1.8.0_161/jre/bin/java -javaagent:/usr/local/src/BurpUnlimited/BurpUnlimited.jar -agentpath:/usr/local/src/BurpUnlimited/lib/libfaketime64.so -jar /usr/local/src/BurpUnlimited/BurpUnlimited.jar



#windows.vbs脚本
set ws=WScript.CreateObject("WScript.Shell") 

ws.Run "burpsuite.bat",0


#windows.bat脚本

cd C:\Program Files\BurpUnlimited
java -javaagent:BurpUnlimited.jar -agentpath:lib/libfaketime64.dll -jar BurpUnlimited.jar
```
下载扩展组件

http://www.jython.org/downloads.html
推荐2个
```
J2EEscan
CO2
```