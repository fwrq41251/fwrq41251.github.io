---
title: linux常用命令
date: 2018-11-14
tags:
  - linux
---

持续更新

#### 查看端口号使用

netstat -tulpn | grep :80

#### 查看硬盘使用

* df -h
* df --inodes

#### 查找进程

ps aux|grep java

#### 创建快捷方式

ln -s /folderorfile/link/will/point/to /name/of/the/link

#### 查找大目录，文件

du -a /home | sort -n -r | head -n 5

#### 查找包含特定文本的文件

grep -rnw '/path/to/somewhere/' -e 'pattern'

### 过滤文本

grep "to-find" 'file' > 'to-file'
