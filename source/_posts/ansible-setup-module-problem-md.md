---
title: ansible的setup模块可能导致进程卡死
date: 2017-01-07 22:11:40
tags:
- ansible
- setup模块
- ansible-playbook
- openstack
categories:
- ansible
- 技术总结
---

## 问题背景

> 概述

一次 playbook 的测试过程中，发现一个奇怪的现象：就是执行 playbook 的过程中，总会出现一个 setup 的步骤，并且这个步骤执行的时间非常的长，达到5分钟，甚至有的环境会直接卡死，进程无法中断。
<!-- more -->
详细的问题描述见[github-ansible](https://github.com/ansible/ansible/issues/19913)

> 环境介绍

通过ansible节点管理openstack环境,所以环境中有很多虚拟网卡，导致setup模块的问题。

## 问题现象

> test.yaml 文件

```yaml
---
- hosts: xx.xx.xx.xx
  tasks:
    - name: test
      shell: echo aaaa
```

执行之后，直接卡死，就像下面这样，一直没有返回。

```bash
# ansible-playbook test.yaml

PLAY [xx.xx.xx.xx] ************************************************************

TASK [setup] *******************************************************************
```

## 问题分析

分别到ansible节点和目标节点查看原因。

> ansible节点

```bash
 7901 pts/2    Sl+    2:43  \_ /usr/bin/python /usr/bin/ansible all -m setup
 7912 pts/2    S+     0:00      \_ /usr/bin/python /usr/bin/ansible all -m setup
 7960 pts/2    S+     0:00          \_ ssh -C -o ControlMaster=auto -o ControlPersist=60s -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o ConnectTimeout=10 -o ControlPath=/root/.ansible/cp/ansible-ssh-%h-%p-%r -tt xx.xx.xx.xx /bin/sh -c '/usr/bin/python /root/.ansible/tmp/ansible-tmp-1483608708.39-143999004080315/setup.py; rm -rf "/root/.ansible/tmp/ansible-tmp-1483608708.39-143999004080315/" > /dev/null 2>&1 && sleep 0'
```

> 目标节点

```bash
49036 pts/1    Ss+    0:00 /bin/sh -c /usr/bin/python /root/.ansible/tmp/ansible-tmp-1483608708.39-143999004080315/setup.py; rm -rf "/root/.ansible/tmp/ansible-tmp-1483608708.39-143999004080315/" > /dev/null 2>&1 && sleep 0
 49083 pts/1    S+     0:00  \_ /usr/bin/python /root/.ansible/tmp/ansible-tmp-1483608708.39-143999004080315/setup.py
 49213 pts/1    D+     0:00      \_ /usr/bin/python /tmp/ansible_bx9h6K/ansible_module_setup.py
```
请注意，这里进程进入了 `D` 状态，这个状态的意思是不可中断的进程，会一直卡着，无法返回。

> setup模块

既然问题出在 setup 模块，那就先来看下setup 模块是干什么的，
通过查看源码和文档，这个是获取系统信息，并将相关变量存储到facts中，具体的就不在这里展开分析了。

其中获取网卡信息，过程大概如下
```bash
80013 pts/2    Ss+    0:00 /bin/sh -c /usr/bin/python /root/.ansible/tmp/ansible-tmp-1483615192.93-195988658363074/setup.py; rm -rf "/root/.ansible/tmp/ansible-tmp-1483615192.93-195988658363074/" > /dev/null 2>&1 && sleep 0
 80034 pts/2    S+     0:00  \_ /usr/bin/python /root/.ansible/tmp/ansible-tmp-1483615192.93-195988658363074/setup.py
 80073 pts/2    S+     0:00      \_ /usr/bin/python /tmp/ansible_pBITNP/ansible_module_setup.py
 80092 pts/2    Sl+    0:25          \_ /usr/bin/ruby /usr/bin/facter --puppet --json
 15676 pts/2    R+     0:00              \_ /sbin/ip link show ovs-system

 80013 pts/2    Ss+    0:00 /bin/sh -c /usr/bin/python /root/.ansible/tmp/ansible-tmp-1483615192.93-195988658363074/setup.py; rm -rf "/root/.ansible/tmp/ansible-tmp-1483615192.93-195988658363074/" > /dev/null 2>&1 && sleep 0
 80034 pts/2    S+     0:00  \_ /usr/bin/python /root/.ansible/tmp/ansible-tmp-1483615192.93-195988658363074/setup.py
 80073 pts/2    S+     0:00      \_ /usr/bin/python /tmp/ansible_pBITNP/ansible_module_setup.py
 80092 pts/2    Sl+    0:25          \_ /usr/bin/ruby /usr/bin/facter --puppet --json
 12971 pts/2    R+     0:00              \_ /sbin/ip link show ifb992
```

## 问题处理

openstack的网络节点，会有非常多的虚拟网卡，我的这个环境中虚拟网卡个数接近1000个了，导致收集信息的时间长。
既然只是获取信息，那就能否跳过这一个步骤呢？当然是有的，修改配置文件 `/etc/ansible/ansible.cfg`，其中有一个配置项 `gather_subset`

```bash
#原值：
#gather_subset=all
#新值：
gather_subset=!all
```
修改之后，再次执行playbook，问题得到解决。具体其他的配置项，可以查看[setup的源码](https://github.com/ansible/ansible/blob/devel/lib/ansible/modules/system/setup.py)。
