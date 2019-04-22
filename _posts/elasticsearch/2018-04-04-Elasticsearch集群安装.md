---
layout: post
title: "Elasticsearch集群安装"
description: Elasticsearch集群安装
category: Elasticsearch
---


# Elasticsearch集群安装

在三台linux服务器上，集群安装ElasticSearch.6.2.2，及其es的插件，各种管理软件

## 环境

域名|ip
---|---
es1|10.131.44.254
es2|172.16.187.143
es3|172.16.185.62

## JDK
三台机器都安装jdk最新版本
 
 ```
[es@es1 ~]$ java -version
java version "1.8.0_161"
Java(TM) SE Runtime Environment (build 1.8.0_161-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.161-b12, mixed mode)
```
## 设置用户

三台机器都统一用户为es

 ```
[root@es1 ~]# useradd es
You have new mail in /var/spool/mail/root
[root@biluos ~]# passwd es                                
Changing password for user es.
New password: 
BAD PASSWORD: it is based on a dictionary word
BAD PASSWORD: is too simple
Retype new password: 
passwd: all authentication tokens updated successfully.
[root@es1 ~]#  mkdir /home/es
mkdir: cannot create directory `/home/es': File exists
[root@es1 ~]#  ll /home/   # 注意是不是es用户和用户组 
total 4
drwx------ 3 es es 4096 Feb 25 03:51 es
 ```
 
 三台机器都建立/data/elasticsearch目录，用来存放es软件包和数据存储，使用es用户
 
```
[root@es1 ~]# su es
[es@es1 ~]$ mkdir -p /data/elasticsearch
```
其余两台此处省略

## 解压

三台机器都解压安装包到/home/es/elasticsearch
[下载](https://www.elastic.co/cn/downloads/elasticsearch)包：elasticsearch-6.2.2.tar.gz

解压：
```
tar -zxvf /home/es/elasticsearch/elasticsearch-6.2.2.tar.gz -C /data/elasticsearch
```

## 修改权限
三台机器都修改es软件包的权限为es用户
```
使用root用户修改权限
[es@es1 ~]$ su root
Password: 
[root@es1 es]# chown -R es:es /data/elasticsearch/
```
其余两台此处省略

三台机器都创建data数据目录和日志目录，使用es用户
```
[root@es1 es]# su es
[es@es1 ~]$ mkdir -p /data/elasticsearch/data/
[es@es1 ~]$ mkdir -p /data/elasticsearch/logs/
```

其余两台此处省略

## 修改es配置

三台机器都修改配置(10.131.44.254)

```
vim /data/elasticsearch/config/elasticsearch.yml

# 集群名称
cluster.name: mz-es

# 日志路径
path.logs: /data/elasticsearch/logs/

# 服务端口
http.port: 9200

# 集群发现 集群节点ip或者主机
discovery.zen.ping.unicast.hosts: ["10.131.44.254", "172.16.187.143","172.16.185.62"]

#设置这个参数来保证集群中的节点可以知道其它N个有master资格的节点。默认为1，对于大的集群来说，可以设置大一点的值（2-4）
discovery.zen.minimum_master_nodes: 2

# 下面两行配置为haad插件配置，三台服务器一致。
http.cors.enabled: true
http.cors.allow-origin: "*"
```
## 修改系统配置

三台机器都修改 Linux下/etc/security/limits.conf文件设置

```
更改linux的最大文件描述限制要求
添加或修改如下：
* soft nofile 262144
* hard nofile 262144 更改linux的锁内存限制要求
添加或修改如下：
es soft memlock unlimited
es hard memlock unlimited 

最后配置如下
# End of file
* soft nofile 262144
* hard nofile 262144
es soft memlock unlimited                                                                                                                                         
es hard memlock unlimited
```
三台机器都修改配置 Linux下/etc/security/limits.d/90-nproc.conf文件设置
更改linux的的最大线程数，添加或修改如下（这里es是es用户）：

```
* soft nproc unlimited
vim /etc/security/limits.d/90-nproc.conf
*          soft    nproc     unlimited
es       soft    nproc     unlimited
```

三台机器都修改配置 Linux下/etc/sysctl.conf文件设置
更改linux一个进行能拥有的最多的内存区域要求,添加或修改如下：
```
vm.max_map_count = 262144 更改linux禁用swapping,添加或修改如下：
vm.swappiness = 1 
vim /etc/sysctl.conf

vm.max_map_count = 262144
vm.swappiness = 1
```

## 启动
```
[es@es1]$ /data/elasticsearch/bin/elasticsearch
[es@es2]$ /data/elasticsearch/bin/elasticsearch
[es@es3]$ /data/elasticsearch/bin/elasticsearch
```

## 结果
```
http://10.131.44.254:9200/

{
    "name": "node-10.131.44.254",
    "cluster_name": "mz-es",
    "cluster_uuid": "ZOW6NjPDRB6fl-dyjXJM5A",
    "version": {
        "number": "6.2.2",
        "build_hash": "10b1edd",
        "build_date": "2018-02-16T19:01:30.685723Z",
        "build_snapshot": false,
        "lucene_version": "7.2.1",
        "minimum_wire_compatibility_version": "5.6.0",
        "minimum_index_compatibility_version": "5.0.0"
    },
    "tagline": "You Know, for Search"
}
```
# ES集成ik分词

## 下载ik安装包

[下载](https://github.com/medcl/elasticsearch-analysis-ik/releases) 版本一定要与ES版本保持一致

## 解压
```
unzip elasticsearch-analysis-ik-6.2.2.zip -d /data/elasticsearch/plugin
```
## 启动es

![image](https://jasperbalcony.github.io/images/elasticsearch/es-ik.png)

## 验证

```
[es@es es]# curl -H 'Content-Type:application/json' 'http://172.16.185.62:9200/uc/_analyze?pretty=true' -d '{"field":"title","text":"我来自山西省"}'

{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<IDEOGRAPHIC>",
      "position" : 0
    },
    {
      "token" : "来",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "<IDEOGRAPHIC>",
      "position" : 1
    },
    {
      "token" : "自",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "<IDEOGRAPHIC>",
      "position" : 2
    },
    {
      "token" : "山",
      "start_offset" : 3,
      "end_offset" : 4,
      "type" : "<IDEOGRAPHIC>",
      "position" : 3
    },
    {
      "token" : "西",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "<IDEOGRAPHIC>",
      "position" : 4
    },
    {
      "token" : "省",
      "start_offset" : 5,
      "end_offset" : 6,
      "type" : "<IDEOGRAPHIC>",
      "position" : 5
    }
  ]
}

```
或者直接借助kibana验证
![image](https://jasperbalcony.github.io/images/elasticsearch/ik-result.jpg)

## 配置自定义分词

### 新建自定义分词
```
/data/elasticsearch/plugins/elasticsearch/config/custom
[es@es1 custom]$ cat new_word.dic 
老铁
王者荣耀
洪荒之力
共有产权房
一带一路
```
### 加入配置
```
[es@es1 config]$ cat IKAnalyzer.cfg.xml 
﻿<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict">custom/new_word.dic</entry>	 
	<!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords"></entry>
	<!--用户可以在这里配置远程扩展字典 -->
	<!-- <entry key="remote_ext_dict">words_location</entry> -->
	<!--用户可以在这里配置远程扩展停止词字典-->
	<!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>
```
### 验证

![image](https://jasperbalcony.github.io/images/elasticsearch/ik-consume.jpg)

# 安装elasticsearch-head

三台机器只需要一台安装head就可以了

采用nodejs安装
6.X中，elasticsearch-head ，不能放在elasticsearch的 plugins、modules 目录下 不能使用 elasticsearch-plugin install 直接启动elasticsearch即可

## 修改配置 
```
elasticsearch/config/elasticsearch.yml

添加
http.cors.enabled: true
http.cors.allow-origin: "*"
```

## 下载安装包 

下载elasticsearch-head 或者 git clone 到随便一个文件夹
https://github.com/mobz/elasticsearch-head

## 安装nodejs

[参考](https://blog.csdn.net/qq_21383435/article/details/79367366)

简单的说 Node.js 就是运行在服务端的 JavaScript。Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行环境。Node.js 使用了一个事件驱动、非阻塞式 I/O 的模型，使其轻量又高效。Node.js 的包管理器 npm，是全球最大的开源库生态系统。

### 下载Node.js

打开官网下载链接:https://nodejs.org/en/download/ 我这里下载的是node-v8.9.4-linux-x64.tar.xz,如下图：

这里写图片描述

![image](https://jasperbalcony.github.io/images/elasticsearch/Node.js.png)

### 解压
```
tar -zxvf node-v8.9.4-linux-x64.tar.xz  -C /opt/moudles/

gzip: stdin: not in gzip format
tar: Child returned status 1
tar: Error is not recoverable: exiting now

tar -xvf node-v8.9.4-linux-x64.tar.xz  -C /opt/moudles/ 

这里-z 用Gzip压缩或解压
```

### 变为全局可用

```
修改 /etc/profile
 vim /etc/profile 

export NODEJS_HOME=/opt/moudles/node-v8.9.4-linux-x64
export PATH=$PATH:$NODEJS_HOME/bin
```
### 测试
```
# node -v               
v8.9.4
```

## 安装elasticsearch-head

```
cd /home/es/elasticsearch-head-master
npm install -g grunt-cli
grunt是一个很方便的构建工具，可以进行打包压缩、测试、执行等等的工作
$ npm install -g grunt-cli 

统一用户为es
[root@es1 es]# chown -R es:es /home/es/elasticsearch-head-master/
然后使用es用户，安装成功
[root@es1 elasticsearch-head-master]# su es 
[es@es1 elasticsearch-head-master]$ npm install 
```
    ^
## 启动

```
$ grunt server &
(node:6612) ExperimentalWarning: The http2 module is an experimental API.
Running "connect:server" (connect) task
Waiting forever...
Started connect web server on https://localhost:9100
```
[查看](http://10.131.44.254:9100/)

![image](https://jasperbalcony.github.io/images/elasticsearch/es-head.jpg)


# 安装kibana

集群的时候，我们只需要在一台机器上安装就可以了，
                                                                                                                                                                                            
## 下载

[下载](https://www.elastic.co/cn/downloads/kibana)  版本要和es版本相同

## 解压
 ```
/home/es/kibana/kibana-6.2.2-linux-x86_64
 ```
 
## 配置
 ```
# vim config/kibana.yml
elasticsearch.url: "http://10.131.44.254:9200/"    # kibana监控哪台es机器
server.host: "10.131.44.254"                # kibana运行在哪台机器
 ```
## 运行
 ```
/data/kibana/bin/kibana &
          
  log   [02:20:13.766] [info][status][plugin:kibana@6.2.2] Status changed from uninitialized to green - Ready
  log   [02:20:13.821] [info][status][plugin:elasticsearch@6.2.2] Status changed from uninitialized to yellow - Waiting for Elasticsearch
  log   [02:20:13.983] [info][status][plugin:timelion@6.2.2] Status changed from uninitialized to green - Ready
  log   [02:20:13.997] [info][status][plugin:console@6.2.2] Status changed from uninitialized to green - Ready
  log   [02:20:14.007] [info][status][plugin:metrics@6.2.2] Status changed from uninitialized to green - Ready
  log   [02:20:14.030] [info][listening] Server running at http://10.131.44.254:5601
  log   [02:20:14.078] [info][status][plugin:elasticsearch@6.2.2] Status changed from yellow to green - Ready
 ```
  查看界面 http://10.131.44.254:5601/ 可以直接访问
  
  ![image](https://jasperbalcony.github.io/images/elasticsearch/kibana.jpg)