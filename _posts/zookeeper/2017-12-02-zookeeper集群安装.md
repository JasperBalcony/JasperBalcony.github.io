---
layout: post
title: "zookeeper集群安装"
description: zookeeper集群安装
category: zookeeper
---

# zookeeper集群安装

## 机器

IP|host
---|---
172.16.185.68|hadoop1
172.16.185.69|hadoop2
72.16.185.70|hadoop3		

### 上传zk安装包

进入官网[下载](http://zookeeper.apache.org/),当前使用 zookeeper-3.4.6.tar.gz 版本

### 解压
```
[hadoop@hadoop1 zookeeper-3.4.6]$ tar -zxvf zookeeper-3.4.6.tar.gz -C /data/
```
### 配置（先在一台节点上配置）
```
3.1添加一个zoo.cfg配置文件
$ZOOKEEPER/conf
mv zoo_sample.cfg zoo.cfg

3.2修改配置文件（zoo.cfg）
dataDir=/data/zookeeper-3.4.6/data

server.1=hadoop1:2888:3888
server.2=hadoop2:2888:3888
server.3=hadoop3:2888:3888

3.3在（dataDir=/data/zookeeper-3.4.6/data）创建一个myid文件，里面内容是server.N中的N（server.2里面内容为2）
echo "1" > myid

3.4将配置好的zk拷贝到其他节点
scp -r /data/zookeeper-3.4.6/ hadoop2:/data/
scp -r /data/zookeeper-3.4.6/ hadoop3:/data/

3.5注意：在其他节点上一定要修改myid的内容
在hadoop2应该讲myid的内容改为2 （echo "2" > /data/zookeeper-3.4.6/data/myid）
在hadoop3应该讲myid的内容改为3 （echo "3" > /data/zookeeper-3.4.6/data/myid）
```
	
### 启动集群

```
分别启动zk
/data/zookeeper-3.4.6/bin/zkServer.sh start
检查启动状态
[hadoop@hadoop1 data]$ /data/zookeeper-3.4.6/bin/zkServer.sh status
JMX enabled by default
Using config: /data/zookeeper-3.4.6/bin/../conf/zoo.cfg
Mode: leader
```

### 本地连接
```
[hadoop@hadoop1 software]$ /data/zookeeper-3.4.6/bin/zkCli.sh 
Connecting to localhost:2181
2018-04-08 15:45:46,561 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.6-1569965, built on 02/20/2014 09:09 GMT
2018-04-08 15:45:46,568 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=hadoop1
2018-04-08 15:45:46,568 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.8.0_144
2018-04-08 15:45:46,571 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
2018-04-08 15:45:46,571 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/data/java/jre
2018-04-08 15:45:46,571 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/data/zookeeper-3.4.6/bin/../build/classes:/data/zookeeper-3.4.6/bin/../build/lib/*.jar:/data/zookeeper-3.4.6/bin/../lib/slf4j-log4j12-1.6.1.jar:/data/zookeeper-3.4.6/bin/../lib/slf4j-api-1.6.1.jar:/data/zookeeper-3.4.6/bin/../lib/netty-3.7.0.Final.jar:/data/zookeeper-3.4.6/bin/../lib/log4j-1.2.16.jar:/data/zookeeper-3.4.6/bin/../lib/jline-0.9.94.jar:/data/zookeeper-3.4.6/bin/../zookeeper-3.4.6.jar:/data/zookeeper-3.4.6/bin/../src/java/lib/*.jar:/data/zookeeper-3.4.6/bin/../conf:.:/data/java/jre/lib/rt.jar:/data/java/lib/dt.jar:/data/java/lib/tools.jar
2018-04-08 15:45:46,572 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
2018-04-08 15:45:46,572 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
2018-04-08 15:45:46,572 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
2018-04-08 15:45:46,572 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
2018-04-08 15:45:46,572 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
2018-04-08 15:45:46,572 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=2.6.32-358.14.1.el6.mz150203.x86_64
2018-04-08 15:45:46,572 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=hadoop
2018-04-08 15:45:46,573 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/home/hadoop
2018-04-08 15:45:46,573 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/data/software
2018-04-08 15:45:46,575 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=localhost:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@799f7e29
Welcome to ZooKeeper!
2018-04-08 15:45:46,630 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@975] - Opening socket connection to server localhost/127.0.0.1:2181. Will not attempt to authenticate using SASL (unknown error)
JLine support is enabled
2018-04-08 15:45:46,719 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@852] - Socket connection established to localhost/127.0.0.1:2181, initiating session
2018-04-08 15:45:46,734 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1235] - Session establishment complete on server localhost/127.0.0.1:2181, sessionid = 0x16270e61c81000d, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2181(CONNECTED) 0] ls /
[zookeeper, yarn-leader-election, hadoop-ha, hbase]
```