---
layout: post
title: "Linux-Mysql5.5.9-1安装"
description: Linux-Mysql5.5.9-1安装
category: 数据库
---


本文介绍的是Linux-Mysql5.5.9-1安装

# 版本

mysql 社区版 5.5.9

```
#添加用户组
groupadd mysql
#添加用户mysql 到用户组mysql
useradd -g mysql mysql
```

# Server
## 下载

```cmd
wget --no-check-certificate https://downloads.mysql.com/archives/get/file/MySQL-server-5.5.9-1.rhel5.x86_64.rpm
```
## 安装

```cmd

# rpm -ivh MySQL-server-5.5.9-1.rhel5.x86_64.rpm

Preparing...                ########################################### [100%]
   1:MySQL-server           ########################################### [100%]

PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
To do so, start the server, then issue the following commands:

/usr/bin/mysqladmin -u root password 'new-password'
/usr/bin/mysqladmin -u root -h GZSB-CJB-SHH1-13-MAEGIS-0.130 password 'new-password'

Alternatively you can run:
/usr/bin/mysql_secure_installation

which will also give you the option of removing the test
databases and anonymous user created by default.  This is
strongly recommended for production servers.

See the manual for more instructions.

Please report any problems with the /usr/bin/mysqlbug script!
```
# client

## 下载
```
wget --no-check-certificate https://downloads.mysql.com/archives/get/file/MySQL-client-5.5.9-1.rhel5.x86_64.rpm
```

## 安装
```
# rpm -ivh MySQL-client-5.5.9-1.rhel5.x86_64.rpm
Preparing...                ########################################### [100%]
   1:MySQL-client           ########################################### [100%]
```
# 启动
```
# service mysql start
Starting MySQL.. SUCCESS! 
```
# 设置密码

```cmd
/usr/bin/mysqladmin -u root password 'jasper'
/usr/bin/mysqladmin -u root -h 127.0.0.1 password 'jasper'
```
# 允许远程访问

```cmd
# mysql -uroot -pjasper

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 5.5.9-log MySQL Community Server (GPL)

Copyright (c) 2000, 2010, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>

输入如下 sql
grant all privileges on *.* to 'root'@'%' identified by 'jasper' with grant option;

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.04 sec)
```
# 配置

## 配置文件
mysql 配置文件是 `/etc/my.cnf`，但是 rpm 方式安装的话，这个文件并不存在，这时只需要将 `/usr/share/mysql/my-medium.cnf` 
复制到 `/etc`下并更名为 `my.cnf` 即可

## 默认字符集
编辑 `/etc/my.cnf`，将默认字符集修改为`utf8mb4`
- 在 [client] 小节，增加 `default-character-set=utf8mb4`
- 在 [mysqld] 小节，增加 `character-set_server=utf8mb4`，注意不要用` default-character-set=utf8mb4`，
否则 mysql 无法启动

## 修改文件路径

- 关闭MySql
```
service mysql stop
```
- 转移数据
```
cd /var/lib
cp -a mysql /data/mysqldata/
```
- 修改配置文件

```
 [client]
  #socket         = /var/lib/mysql/mysql.sock
  socket          = /data/mysqldata/mysql/mysql.sock
  
 [mysqld]
  #socket         = /var/lib/mysql/mysql.sock
  socket          = /data/mysqldata/mysql/mysql.sock
  datadir=/data/mysqldata/mysql   
```

- 启动MySql

```
service mysql start
```

这样以后mysql文件全部转移到 `/data/mysqldata/mysql` 目录