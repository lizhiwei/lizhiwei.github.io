<!--
author: lizhiwei
head: 
date: 2019-09-09
title: Android 5.0之后反向usb网络共享
tags: https,sniffer,mitm
images: 
category: network
status: publish
summary: 在Android 5.0之后引入了ip rule,使得路由规则更复杂，之间的reverse tethering方法需要进行修改
-->


## 什么是反向USB网络共享

### USB网络共享

手机能通过移动数据网络上网，通过usb线缆，将网络共享给PC

### 反向USB网络共享

PC能通过局域网（有线或者无线), 通过USＢ线缆，将网络共享给手机
手机无需插入sim，也无须开启WiFI


### Android Lollipop(5.0)之后引入了ip rule规则

   使得路由规则更复杂，以前的反向usb网络共享的方法，需要进行调整


## 操作步骤

### 手机上　开启　无线数据网络 或者　wifi

#### ifconfig 可看到ip分配情况

    rmnet_data3 Link encap:UNSPEC  
          inet addr:10.24.148.231  Mask:255.255.255.240 
          inet6 addr: 2408:84f1:ffb5:1663:5c21:9cff:fec7:f70b/64 Scope: Global
          inet6 addr: fe80::5c21:9cff:fec7:f70b/64 Scope: Link
          UP RUNNING  MTU:1500  Metric:1
          RX packets:1058 errors:0 dropped:0 overruns:0 frame:0 
          TX packets:1061 errors:0 dropped:0 overruns:0 carrier:0 
          collisions:0 txqueuelen:1000 
          RX bytes:399777 TX bytes:126003 


#### ip rule show 可看到路由规则

    0:	from all lookup local 
    10000:	from all fwmark 0xc0000/0xd0000 lookup legacy_system 
    10500:	from all iif lo oif dummy0 uidrange 0-0 lookup dummy0 
    10500:	from all iif lo oif rmnet_data0 uidrange 0-0 lookup rmnet_data0 
    10500:	from all iif lo oif rmnet_data1 uidrange 0-0 lookup rmnet_data1 
    10500:	from all iif lo oif rmnet_data3 uidrange 0-0 lookup rmnet_data3 
    10500:	from all iif lo oif rmnet_data2 uidrange 0-0 lookup rmnet_data2 
    13000:	from all fwmark 0x10063/0x1ffff iif lo lookup local_network 
    13000:	from all fwmark 0xd0001/0xdffff iif lo lookup rmnet_data0 
    13000:	from all fwmark 0xd0064/0xdffff iif lo lookup rmnet_data1 
    13000:	from all fwmark 0x10065/0x1ffff iif lo lookup rmnet_data3 
    13000:	from all fwmark 0xd0066/0xdffff iif lo lookup rmnet_data2 
    14000:	from all iif lo oif dummy0 lookup dummy0 
    14000:	from all fwmark 0xc0000/0xc0000 iif lo oif rmnet_data0 lookup rmnet_data0 
    14000:	from all fwmark 0xc0000/0xc0000 iif lo oif rmnet_data1 lookup rmnet_data1 
    14000:	from all iif lo oif rmnet_data3 lookup rmnet_data3 
    14000:	from all fwmark 0xc0000/0xc0000 iif lo oif rmnet_data2 lookup rmnet_data2 
    15000:	from all fwmark 0x0/0x10000 lookup legacy_system 
    16000:	from all fwmark 0x0/0x10000 lookup legacy_network 
    17000:	from all fwmark 0x0/0x10000 lookup local_network 
    19000:	from all fwmark 0x65/0x1ffff iif lo lookup rmnet_data3 
    22000:	from all fwmark 0x0/0xffff iif lo lookup rmnet_data3 
    32000:	from all unreachable

### 开启USB网络共享

#### 可以看到分配的IP

    rndis0    Link encap:Ethernet  HWaddr 62:b3:d8:11:39:6d
          inet addr:192.168.42.129  Bcast:192.168.42.255  Mask:255.255.255.0 
          inet6 addr: fe80::60b3:d8ff:fe11:396d/64 Scope: Link
          inet6 addr: 2408:84f1:fff7:6c57::35/64 Scope: Global
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0 
          TX packets:10 errors:0 dropped:0 overruns:0 carrier:0 
          collisions:0 txqueuelen:1000 
          RX bytes:0 TX bytes:1852 

#### 新加的ip rule 

    0:	from all lookup local 
    10000:	from all fwmark 0xc0000/0xd0000 lookup legacy_system 
    10500:	from all iif lo oif dummy0 uidrange 0-0 lookup dummy0 
    10500:	from all iif lo oif rmnet_data0 uidrange 0-0 lookup rmnet_data0 
    10500:	from all iif lo oif rmnet_data1 uidrange 0-0 lookup rmnet_data1 
    10500:	from all iif lo oif rmnet_data3 uidrange 0-0 lookup rmnet_data3 
    10500:	from all iif lo oif rmnet_data2 uidrange 0-0 lookup rmnet_data2 
         10500:	from all iif lo oif rndis0 uidrange 0-0 lookup local_network 
    13000:	from all fwmark 0x10063/0x1ffff iif lo lookup local_network 
    13000:	from all fwmark 0xd0001/0xdffff iif lo lookup rmnet_data0 
    13000:	from all fwmark 0xd0064/0xdffff iif lo lookup rmnet_data1 
    13000:	from all fwmark 0x10065/0x1ffff iif lo lookup rmnet_data3 
    13000:	from all fwmark 0xd0066/0xdffff iif lo lookup rmnet_data2 
    14000:	from all iif lo oif dummy0 lookup dummy0 
    14000:	from all fwmark 0xc0000/0xc0000 iif lo oif rmnet_data0 lookup rmnet_data0 
    14000:	from all fwmark 0xc0000/0xc0000 iif lo oif rmnet_data1 lookup rmnet_data1 
    14000:	from all iif lo oif rmnet_data3 lookup rmnet_data3 
    14000:	from all fwmark 0xc0000/0xc0000 iif lo oif rmnet_data2 lookup rmnet_data2 
        14000:	from all iif lo oif rndis0 lookup local_network 
    15000:	from all fwmark 0x0/0x10000 lookup legacy_system 
    16000:	from all fwmark 0x0/0x10000 lookup legacy_network 
    17000:	from all fwmark 0x0/0x10000 lookup local_network 
       18000:	from all iif rndis0 lookup rmnet_data3 
    19000:	from all fwmark 0x65/0x1ffff iif lo lookup rmnet_data3 
    22000:	from all fwmark 0x0/0xffff iif lo lookup rmnet_data3 
    32000:	from all unreachable

#### 改变手机上rndis0接口分配的IP

    ifconfig rndis0  10.42.0.2 netmask 255.255.255.0

#### 修改　路由表　　rmnet_data3

    ip route add 10.42.0.0/24 dev rndis0 table rmnet_data3
    ip route delete default table rmnet_data3
    ip route add default via 10.42.0.1 table rmnet_data3


#### 删除ip v6的表　（选用)
　　　防止走ipv6出去

    ip -6 route  del default  table rmnet_data3



### 　PC上的修改
　
####  开启IP转发

    echo 1 > /proc/sys/net/ipv4/ip_forward

#### 开启地址转换

    iptables -t nat -A POSTROUTING -j MASQUERADE

    iptables -t nat -n -L


#### 给USB网卡分配IP地址

    ip addr add 10.42.0.1/24 dev enp56s0u2



###  在手机上检测
　　　
　   ping 10.42.0.1
    PING 10.42.0.1 (10.42.0.1) 56(84) bytes of data.
    64 bytes from 10.42.0.1: icmp_seq=1 ttl=64 time=0.985 ms

　　ping www.baidu.com
    
### 设置DNS


    ndc resolver setnetdns rmnet_data3 114.114.114.114 114.114.115.115
[Android手机上如何修改移动数据网络3G4G5G的DNS](Android手机上如何修改移动数据网络3G4G5G的DNS.html)

