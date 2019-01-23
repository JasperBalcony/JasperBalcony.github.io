---
layout: post
title: "VMware NAT方式创建虚拟机网络并配置固定IP"
description: VMware NAT方式创建虚拟机网络并配置固定IP
category: Linux
---

# 环境

- VMware Workstation 10

- 本机：Windows8 64位，能够访问网络，自动获得IP地址和DNS服务器地址

- 虚拟机：CentOS-6.4-x86_64-bin-DVD1.iso

# 实现目标
- 使用NAT方式创建虚拟机网络，主机IP与虚拟机IP相互ping通

- 虚拟机能够访问网络

- 三台虚拟机IP固定为 192.168.137.100、192.168.137.101、192.168.137.102

# 步骤
本机共享网络给虚拟机VMnet8网卡：本机网卡右键“属性”-->“共享”-->勾选“允许其他网络用户通过此计算机的Internet连接来连接(N)”-->选择虚拟机网卡VMnet8

![image](https://jasperbalcony.github.io/images/vm/vm-1.png)

之后系统会提示VMnet8网卡的IP将变成192.168.137.1，确认即可。图中的VMnet8为已启用状态，如果显示未识别的网络，请参考VMnet1和VMnet8 未识别的网络的解决方法

编辑虚拟机网卡VMnet8属性：VMnet8网卡右键“属性”-->双击“Internet协议版本4(TCP/IPv4)”-->IP地址和DNS地址使用图中配置

![image](https://jasperbalcony.github.io/images/vm/vm-2.png)

编辑VMware Workstation虚拟网络配置：“编辑”-->“虚拟网络编辑器”-->选中VMnet8网卡-->点击“NAT设置(S)”-->设置“网关IP(G)”

![image](https://jasperbalcony.github.io/images/vm/vm-3.png)

![image](https://jasperbalcony.github.io/images/vm/vm-4.png)

![image](https://jasperbalcony.github.io/images/vm/vm-5.png)

创建虚拟机：网络适配器选择NAT模式

![image](https://jasperbalcony.github.io/images/vm/vm-6.png)


编辑虚拟机网卡配置文件(文件名以实际为准)：NETMASK、DNS1、DNS2为第二步中配置的IP；GATEWAY为第三步中配置的IP；IPADDR为虚拟机使用的固定IP

具体修改如下
```
DEVICE=eth0
HWADDR=00:0C:29:C8:0C:94
TYPE=Ethernet
UUID=9ea2946e-bd08-4f3d-9dc6-b2ea48243d3e
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=static
IPADDR=192.168.137.100
NETMASK=255.255.255.0
GATEWAY=192.168.137.2
DNS1=8.8.8.8
DNS2=114.114.114.114
```


重启虚拟机网卡：

```
# service network restart
Shutting down interface eth0:                              [  OK  ]
Shutting down loopback interface:                          [  OK  ]
Bringing up loopback interface:                            [  OK  ]
Bringing up interface eth0:                                [  OK  ]

```

配置其余两台192.168.137.101、192.168.137.102配置

- 克隆192.168.137.100

安装完一个centos虚拟机，又拷贝一份，开机后网卡无法正常启动，报错：`Device eth0 does not seem to be present, 
delaying initialization`

解决：

修改网卡名称
```
# mv /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-eth1
```

```
vim /etc/sysconfig/network-scripts/ifcfg-eth1
```
修改DEVICE="eth0" 为DEVICE="eth1"，可删掉uuid、物理地址

具体修改如下
```
DEVICE=eth1
#HWADDR=00:0C:29:C8:0C:94
TYPE=Ethernet
#UUID=9ea2946e-bd08-4f3d-9dc6-b2ea48243d3e
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=static
IPADDR=192.168.137.101
NETMASK=255.255.255.0
GATEWAY=192.168.137.2
DNS1=8.8.8.8
DNS2=114.114.114.114
```

# 检验

ifconfig指令查看虚拟机IP是否为192.168.137.100

```
# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:0C:29:C8:0C:94  
          inet addr:192.168.137.100  Bcast:192.168.137.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fec8:c94/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:714 errors:0 dropped:0 overruns:0 frame:0
          TX packets:406 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:69810 (68.1 KiB)  TX bytes:50292 (49.1 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)
```

本机IP和虚拟机IP查看是否能互相ping通

ping www.baidu.com指令查看是否能ping通
```
# ping www.baidu.com
PING www.wshifen.com (104.193.88.77) 56(84) bytes of data.
64 bytes from 104.193.88.77: icmp_seq=1 ttl=128 time=195 ms
64 bytes from 104.193.88.77: icmp_seq=2 ttl=128 time=209 ms
64 bytes from 104.193.88.77: icmp_seq=3 ttl=128 time=209 ms
64 bytes from 104.193.88.77: icmp_seq=4 ttl=128 time=206 ms
64 bytes from 104.193.88.77: icmp_seq=5 ttl=128 time=214 ms
^C
--- www.wshifen.com ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4610ms
rtt min/avg/max/mdev = 195.671/207.192/214.630/6.377 ms

```

# Xshell连接

![image](https://jasperbalcony.github.io/images/vm/vm-7.png)

参考

[VMware NAT方式创建虚拟机网络并配置固定IP](https://segmentfault.com/a/1190000008743806)