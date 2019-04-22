---
layout: post
title: "skywalking集群部署及业务接入"
description: skywalking集群部署及业务接入
category: skywalking
---

# skywalking集群部署及业务接入

## 容量规划

IP|zk|es|sw
---|---|---|---
10.21.12.64|zookeeper|node-64|OAPServer、skywalking-webapp
10.21.12.65|zookeeper|node-65|OAPServer
10.21.12.66|zookeeper|node-66|


## 安装ZooKeeper

### 下载

进入官网[下载](http://zookeeper.apache.org/),当前使用 zookeeper-3.5.4-beta.tar.gz 版本

### 解压

```
tar -zxvf zookeeper-3.5.4-beta.tar.gz -C /data/
```

### 配置（先在一台节点上配置）

```
# 添加一个zoo.cfg配置文件
$ZOOKEEPER/conf
mv zoo_sample.cfg zoo.cfg

# 修改配置文件（zoo.cfg）
dataDir=/data/zookeeper-3.4.6/data

server.1=10.21.12.64:2888:3888
server.2=10.21.12.65:2888:3888
server.3=10.21.12.66:2888:3888

# 在（dataDir=/data/zookeeper/data）创建一个myid文件，里面内容是server.N中的N（server.2里面内容为2）
echo "1" > myid

# 将配置好的zk拷贝到其他节点

# 注意：在其他节点上一定要修改myid的内容
在10.21.12.65应该讲myid的内容改为2 （echo "2" > /data/zookeeper/data/myid）
在10.21.12.66应该讲myid的内容改为3 （echo "3" > /data/zookeeper/data/myid）
```
	
### 启动集群

```
分别启动zk
/data/zookeeper/bin/zkServer.sh start

检查启动状态
/data/zookeeper-3.4.6/bin/zkServer.sh status
JMX enabled by default
Using config: /data/zookeeper-3.4.6/bin/../conf/zoo.cfg
Mode: leader
```

### 连接

```
/data/zookeeper/bin/zkCli.sh
[zk: localhost:2181(CONNECTED) 2] ls /skywalking/sw/remote
[81d77f0b-da8e-44ac-815f-b3bdc389877a, e7094b12-7b76-4c8e-8ae0-f4adbb28264e
```


## 安装elasticsearch

### 下载

```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.5.4.tar.gz
```
### 设置用户

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

### 解压

三台机器都解压安装包到/home/es/elasticsearch
[下载](https://www.elastic.co/cn/downloads/elasticsearch)包：elasticsearch-6.5.4.tar.gz

解压：

```
tar -zxvf /home/es/elasticsearch/elasticsearch-6.5.4.tar.gz -C /data/elasticsearch
```

### 修改权限
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

### 修改es配置

三台机器都修改配置

```
vim /data/elasticsearch/config/elasticsearch.yml

# 集群名称
cluster.name: skywalking-es

# 日志路径
path.logs: /data/elasticsearch/logs/

# 服务端口
http.port: 9200

# 集群发现 集群节点ip或者主机
discovery.zen.ping.unicast.hosts: ["10.21.12.64", "10.21.12.65" ,"10.21.12.66"]

#设置这个参数来保证集群中的节点可以知道其它N个有master资格的节点。默认为1，对于大的集群来说，可以设置大一点的值（2-4）
discovery.zen.minimum_master_nodes: 2

# 下面两行配置为haad插件配置，三台服务器一致。
http.cors.enabled: true
http.cors.allow-origin: "*"
```
### 修改系统配置

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

### 启动

```
[es@es1]$ /data/elasticsearch/bin/elasticsearch -d
[es@es2]$ /data/elasticsearch/bin/elasticsearch -d
[es@es3]$ /data/elasticsearch/bin/elasticsearch -d
```

### 结果

- http://10.21.12.64:9200/

```
{
    "name": "node-64",
    "cluster_name": "skywalking-es",
    "cluster_uuid": "2JLf_nL6RnixLkrQ4pD03Q",
    "version": {
        "number": "6.5.4",
        "build_flavor": "default",
        "build_type": "tar",
        "build_hash": "d2ef93d",
        "build_date": "2018-12-17T21:17:40.758843Z",
        "build_snapshot": false,
        "lucene_version": "7.5.0",
        "minimum_wire_compatibility_version": "5.6.0",
        "minimum_index_compatibility_version": "5.0.0"
    },
    "tagline": "You Know, for Search"
}
```
- http://10.21.12.64:9200/_cat/nodes?v

```
ip          heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
10.21.12.66           34          21   3    0.96    1.01     1.04 mdi       -      node-66
10.21.12.65           36          98   0    0.87    0.63     0.42 mdi       -      node-65
10.21.12.64           39          45   0    0.14    0.13     0.14 mdi       *      node-64
```

- http://10.21.12.64:9200/_cat/shards?v

```
index                                      shard prirep state   docs   store ip          node
sw_instance_jvm_memory_heap_max_month      1     p      STARTED    1   4.1kb 10.21.12.65 node-65
sw_instance_jvm_memory_heap_max_month      0     p      STARTED    2     8kb 10.21.12.66 node-66
sw_endpoint_relation_cpm                   1     p      STARTED    0    261b 10.21.12.66 node-66
sw_endpoint_relation_cpm                   0     p      STARTED    0    261b 10.21.12.64 node-64
sw_instance_jvm_old_gc_time_hour           1     p      STARTED    3  12.1kb 10.21.12.66 node-66
sw_instance_jvm_old_gc_time_hour           0     p      STARTED    3  11.8kb 10.21.12.64 node-64
sw_service_p90_month                       1     p      STARTED    0    261b 10.21.12.65 node-65
sw_service_p90_month                       0     p      STARTED    0    261b 10.21.12.66 node-66
sw_service_instance_sla_month              1     p      STARTED    0    261b 10.21.12.65 node-65
sw_service_instance_sla_month              0     p      STARTED    0    261b 10.21.12.66 node-66
sw_service_p90                             1     p      STARTED    0    261b 10.21.12.65 node-65
sw_service_p90                             0     p      STARTED    0    261b 10.21.12.66 node-66
sw_endpoint_p95_month                      1     p      STARTED    0    261b 10.21.12.65 node-65
sw_endpoint_p95_month                      0     p      STARTED    0    261b 10.21.12.66 node-66
sw_endpoint_cpm_month                      1     p      STARTED    0    261b 10.21.12.65 node-65
sw_endpoint_cpm_month                      0     p      STARTED    0    261b 10.21.12.66 node-66
sw_service_instance_cpm                    1     p      STARTED    0    261b 10.21.12.64 node-64
sw_service_instance_cpm                    0     p      STARTED    0    261b 10.21.12.65 node-65
sw_service_p95                             1     p      STARTED    0    261b 10.21.12.66 node-66
```

## 安装skywalking
## 下载

```
wget https://www.apache.org/dyn/closer.cgi/incubator/skywalking/6.0.0-GA/apache-skywalking-apm-incubating-6.0.0-GA.tar.gz
```

## 解压

```
tar -zxvf /data/software/apache-skywalking-apm-incubating-6.0.0-GA.tar.gz -C /data/
```

## 修改配置

```
/data/skywalking/config/application.yml
```

修改zk配置cluster.zookeeper 和 storage.elasticsearch集群配置

## 启动

```
# 10.21.12.64 启动OAPServer、skywalking-webapp进程
/data/skywalking/bin/startup.sh
# 10.21.12.65 启动OAPServer进程
/data/skywalking/bin/oapService.sh
```
## 查看

http://10.21.12.64:8080   admin/admin

# 接入

## 复制agent

```
cp -r /data/skywalking/agent/ /data/sky_agent/
```
## 修改配置

/data/sky_agent/agent/config/agent.config

```
agent.service_name=${SW_AGENT_NAME:predictor-serving}
collector.backend_service=${SW_AGENT_COLLECTOR_BACKEND_SERVICES:10.21.12.64:11800,10.21.12.65:11800}
```
### 采样配置

```
agent.sample_n_per_3_secs=${SW_AGENT_SAMPLE:1000}
```

skywalking 全自动探针监控，不需要修改应用程序代码

高性能探针，针对单实例5000tps的应用，在全量采集的情况下，只增加 10% 的CPU开销。换成取样数来计算，`SAMPLE_N_PER_3_SECS = 15000（5000 * 3 ）` 只增加 10% 的CPU开销。

将取样率设置为` SAMPLE_N_PER_3_SECS = 1500`  预计大约会增加 1% 的CPU开销。

那么，具体值视系统或服务的并发情况，可在测试环境下取得经验值的尝试范围将控制在[500 - 1500]即可


## 手动探针

- 引入依赖

```xml
<!--手动探针-->
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-trace</artifactId>
    <version>6.0.0-GA</version>
</dependency>
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-opentracing</artifactId>
    <version>6.0.0-GA</version>
</dependency>
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-log4j-1.x</artifactId>
    <version>6.0.0-GA</version>
</dependency>
<!--手动探针-->
```

- 自行需要埋点方法中加入 `@Trace`

```java
    @RequestMapping("/hello")
    public String index() throws InterruptedException {
        functionA();
        functionB();
        functionC();
        return "Hello World";
    }

    @Trace
    private void functionA() throws InterruptedException {
        long rangeLong = new RandomDataGenerator().nextLong(100, 200);
        Thread.sleep(rangeLong);
        LOGGER.info("functionA traceId:{} use:{} ms", TraceContext.traceId(), rangeLong);
        //在被追踪的方法中自定义 tag.
        ActiveSpan.tag("functionA_tag", "exec functionA");
        functionA_1();
    }

    @Trace
    private void functionA_1() throws InterruptedException {
        long rangeLong = new RandomDataGenerator().nextLong(100, 200);
        Thread.sleep(rangeLong);
        LOGGER.info("functionA_1 traceId:{} use:{} ms", TraceContext.traceId(), rangeLong);
        ActiveSpan.tag("functionA_1_tag", "exec functionA_1");

        functionA_1_1();
        functionA_1_2();
    }
```

## 启动agent

jvm 启动参数增加agent

```
-javaagent:/data/sky_agent/agent/skywalking-agent.jar
```

![image](https://jasperbalcony.github.io/images/sw/sw-1.jpg)

![image](https://jasperbalcony.github.io/images/sw/sw-2.jpg)
