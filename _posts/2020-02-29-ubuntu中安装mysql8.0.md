---
layout: post
title: ubuntu中安装mysql8.0
excerpt: ubuntu中安装mysql8.0
categories: 数据库
keywords: mysql
---
本文记录mysql8.0安装过程
## 安装步骤
- cd /usr/local
- 下载仓储：wget -c https://repo.mysql.com//mysql-apt-config_0.8.15-1_all.deb
- 安装仓储：sudo dpkg -i mysql-apt-config_0.8.15-1_all.deb
- 更新仓储：sudo apt update
- 安装mysql：sudo apt-get install mysql-server
- 配置策略：sudo mysql_secure_installation，这里设置允许远程连接
- 配置开机启动：sudo systemctl enable mysql
- 配置远程连接：修改下root的host为%
    - update user set host='%' where user='root';

## 其他
- 在这个网址可以查看最新 版本:https://dev.mysql.com/downloads/repo/apt/
