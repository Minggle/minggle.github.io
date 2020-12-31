---
layout: post
title: CentOS history 添加时间戳
categories: CentOS
description: CentOS history 添加时间戳
keywords: centos, history, 时间戳
---

- 设置history时间戳

```shell
[root@localhost ~]# echo -e "export HISTSIZE=9999\nexport HISTTIMEFORMAT=\"%F %T \"\nexport PROMPT_COMMAND=\"history -a; \$PROMPT_COMMAND\"" >> /etc/bashrc
[root@localhost ~]# source /root/.bashrc
```

- 查看配置

```shell
[root@localhost ~]# tail -3 /etc/bashrc 
export HISTSIZE=9999 #history 最大行数
export HISTTIMEFORMAT="%F %T " # 添加时间戳
export PROMPT_COMMAND="history -a; $PROMPT_COMMAND" # 添加bash_history时间戳
```

