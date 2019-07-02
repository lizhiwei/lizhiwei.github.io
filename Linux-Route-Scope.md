<!--
author: lizhiwei
head: 
date: 2019-05-04
title: Linux中IP地址Scope
tags: IPv6
images: 
category: network
status: publish
summary: IP地址的范围
-->


## 地址范围
一般有  host   link   global 这几个范围

在Linux中，路由的范围，用来指示 目标网络的距离：

- Host: 本地主机范围，在这台机器内部有效。只能用于本机内通信
例如：loopback地址127.0.0.1 就属于Host范围

- Link：本地网络范围，只在这台设备上有效，只能用于局域网。
从link-local地址来的包，不会被路由器转发
当dhcp失败时，会配置169.254/16地址

因为不会被路由器转发，在LAN同一个交换机下，是可以直接访问的，也就是direct route的概念



- Universe(Global)： 全局范围，超过1跳。

- Site: IPV6环境下，在该站点内有效
路由器也不会转发site-local地址来的包




scope这个概念，主要用在一个主机有多个接口时，决定用哪个地址跟别人通信。
它想同自己主机上服务通信时，就应该使用loopback地址


##  多播地址

1.FF02::1  

链路上的所有IPv6节点

2.FF02::2  

链路上的所有IPv6路由器

路由征集广播，是发送给 FF02::2 的

3.FF02::16  

MLDv2报告 （定义在RFC 3810)  多播监听报告消息

4.ff02::1:ffxx:xxxx   

Solicited Node Address (RFC4291) 

通过将Solicited-node multicast地址与IPv6 Neighbor Discovery协议相结合，能够有效减少广播范围。Solicited-Node multicast address由前缀FF02:0:0:0:0:1:FF00::/104及Unicast地址或Anycast地址的最后24位产生

所以，某主机，用自己的 IP地址的后24 bits, 生成特定的 多播目标地址
 发送 邻居征集请求， 以方便邻居们 回复自己




5.ff02::1:2   

所有的DHCP代理和服务器

客户端的 DHCPv6 请求，是发给这个多播地址的

具体过程参考 [邻居发现协议中关于DHCPv6的过程](/blog/IPv6-NDP.html)



