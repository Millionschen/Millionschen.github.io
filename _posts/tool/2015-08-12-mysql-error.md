---
layout: post
title: mysql安装错误解决办法
category: 工具
tags: Essay
---

## 问题
start: Job failed to start
invoke-rc.d: initscript mysql, action "start" failed.
dpkg: error processing mysql-server-5.5 (--configure):
subprocess installed post-installation script returned error exit status 1
dpkg: dependency problems prevent configuration of mysql-server:
mysql-server depends on mysql-server-5.5; however:
Package mysql-server-5.5 is not configured yet.
dpkg: error processing mysql-server (--configure):
dependency problems - leaving unconfigured
No apport report written because the error message indicates its a followup error from a previous failure.
Errors were encountered while processing:

此时,删除 /var/lib/mysql 以及 /etc/mysql
而后卸载掉mysql相关软件,重新安装就可以了
sudo apt-get autoremove mysql* --purge
sudo apt-get install mysql-server mysql-common



