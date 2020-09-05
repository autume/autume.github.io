---
layout: post
title: ubuntu中mysql安装及简单配置(在线安装方式)
excerpt: ubuntu中mysql安装及简单配置
categories: MySQL
keywords: mysql 数据库
---
本文使用的版本为5.5.62或5.7.29，使用apt方式直接进行安装。
## 安装
- 执行命令:sudo apt-get install mysql-server
- 查看是否安装成功:sudo netstat -tap | grep mysql
- 服务启动后端口查询:sudo netstat -anp | grep mysql

## 跳过MySQL的密码认证过程,重置密码
- 进入vim /etc/my.cnf 或者 /etc/mysql/mysql.conf.d/mysqld.cnf
- 在[mysqld]后面任意一行添加“skip-grant-tables”用来跳过密码验证的过程
- 重启MySQL:sudo service mysql restart 或者/etc/init.d/mysql restart(有些用户可能需要使用/etc/init.d/mysqld restart)
- 输入mysql进入修改密码
- update user set authentication_string=password("oden123") where user="root";
- 执行flush privileges 命令
- 使用密码登录:mysql -u root -p
- 编辑my.cnf,去掉刚才添加的内容，然后重启MySQL

## 创建新用户
- create user 'oden'@'localhost';
- set password for 'oden'@'localhost' = password('123456');

## 配置新用户权限
- grant all privileges on testDb.* to 'oden'@'localhost':将testDb的全部权限赋予用户oden

## 配置允许远程访问
- 在my.cnf或者mysqld.cnf中将bind-address屏蔽或修改成: bind-address = 0.0.0.0
- 进入mysql，执行: update user set host='%' where user='root';
- grant all privileges on \*.\* to 'root'@'%' identified by 'ode123' WITH GRANT OPTION;
	- all PRIVILEGES 表示赋予所有的权限给指定用户，这里也可以替换为赋予某一具体的权限，例如：select,insert,update,delete,create,drop 等，具体权限间用“,”半角逗号分隔
- flush privileges

## 配置管理
- 通过这种方式安装好之后开机自启动都已经配置好,安装好之后会创建如下目录
	- 数据库目录：/var/lib/mysql/
	- 配置文件：/usr/share/mysql（命令及配置文件） ，/etc/mysql（如：my.cnf）
	- 相关命令：/usr/bin(mysqladmin mysqldump等命令) 和/usr/sbin
	- 启动脚本：/etc/init.d/mysql（启动脚本文件mysql的目录）

## 其他
- mysql 新设置用户或更改密码后需用flush privileges刷新MySQL的系统权限相关表，否则会出现拒绝访问，还有一种方法，就是重新启动mysql服务器，来使新设置生效。

## cli命令
- use dbName：选择数据库

## 卸载
- 删除mysql的数据文件:sudo rm /var/lib/mysql/ -R
- 删除mysql的配置文件:sudo rm /etc/mysql/ -R
- 自动卸载mysql（包括server和client）:
    - sudo apt-get autoremove mysql* --purge
    - sudo apt-get remove apparmor
- 在终端中查看MySQL的依赖项：dpkg --list|grep mysql
- 删除依赖项： sudo apt-get remove dbconfig-mysql
- 如果重新安装出现问题的话，这里重启下服务器可能能解决
## 常用命令
- sudo service mysql restar
- sudo service mysql start
- sudo service mysql stop
- sudo service mysql status
- 查看用户及权限：SELECT DISTINCT CONCAT('User: ''',user,'''@''',host,''';') AS query FROM mysql.user;或selsec user,host from mysql.user;
- 开放端口:sudo ufw allow 3306
- 关掉防火墙:sudo ufw disable
- 启动防火墙:sudo ufw enable
- 查看防火墙状态:sudo ufw status
- 配置文件路径在不同版本下可能不同
	- 5.5.62：/etc/my.cnf
	- 5.7.29： /etc/mysql/mysql.conf.d/mysqld.cnf
