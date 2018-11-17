---
layout: post
title: "Windows系统安装MySQL-8.0.12"
description: Windows系统安装MySQL-8.0.12
category: 数据库
---


本文介绍的是MySQL-8.0.12 windows安装

# 下载

[官网](https://dev.mysql.com/downloads/mysql) 或者 [百度网盘](https://pan.baidu.com/s/1OPJojkcvPP5kr7roKrYYRQ)

# 安装

下载完成，将文件解压到你想要安装的盘里。这里我安装到了D盘。之后以管理员身份运行DOS窗口。
进入到mysql的bin文件夹

```cmd
D:\mysql-8.0.12-winx64\bin>
```

## my.ini

- 在mysql-8.0.12-winx64的文件夹下创建一个名为data的空文件夹。
- bin目录中创建一个my.ini的文件，内容为

```
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8mb4
[mysqld]
#设置3306端口
port = 3306 
# 设置mysql的安装目录
basedir=D:/mysql-8.0.12-winx64
# 设置mysql数据库的数据的存放目录
datadir=D:/mysql-8.0.12-winx64/data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
default_authentication_plugin=mysql_native_password
```

## 执行命令

依次执行安装命令 `mysqld --initialize-insecure`、`mysqld -install`
```cmd
D:\mysql-8.0.12-winx64\bin>mysqld --initialize-insecure

D:\mysql-8.0.12-winx64\bin>mysqld -install
Service successfully installed.

```

启动服务 `net start mysql`

```
D:\mysql-8.0.12-winx64\bin>net start mysql
MySQL 服务正在启动 ....
MySQL 服务已经启动成功。

```

此时mysql没有密码，需要进行设置密码，输入：mysqladmin -u root password root
```
D:\mysql-8.0.12-winx64\bin>mysqladmin -u root password root
mysqladmin: [Warning] Using a password on the command line interface can be insecure.
Warning: Since password will be sent to server in plain text, use ssl connection to ensure password safety.
```

登陆

```
D:\mysql-8.0.12-winx64\bin>mysql -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.12 MySQL Community Server - GPL

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql>

```

进行授权远程连接
```
mysql> use mysql;
Database changed

mysql> update user set host = '%' where user = 'root';
Query OK, 1 row affected (0.10 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.04 sec)
```
由于8.0以上用户密码方式为caching_sha2_password，Navicat无法登陆，更改ROOT用户的native_password密码
_**mysql8.0以上密码策略限制必须要大小写加数字特殊符号**_

```
mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'jasper@006';
Query OK, 0 rows affected (0.15 sec)
```
Navicat连接

![image](https://jasperxgwang.github.io/images/mysql/navicat.jpg)




