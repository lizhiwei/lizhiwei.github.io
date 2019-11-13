<!--
author: lizhiwei
head: 
date: 2019-11-11
title: OpenWRT将router改成switch加ap模式
tags: IPv6
images: 
category: network
status: publish
summary: OpenWRT将router改成switch加ap模式
-->


## 准备材料
  1. 两台智博通 WG3526 路由器 (MT7621A 16M ROM,  512M RAM)
  2. 中国联通FTTH
  

## 其中一台改为交换机模式



### 修改LAN配置
/etc/config/network

    config interface 'lan'
    	option type 'bridge'
    	option ifname 'eth0.1'
    	option proto 'static'
    	option ipaddr '192.168.0.2'
    	option netmask '255.255.255.0'
    	option ip6assign '60'


其中ip地址改为 192.168.0.2 避免跟主路由器冲突

### 关闭wan
/etc/config/network

       #config interface 'wan'
       #       option ifname 'eth0.2'
       #       option proto 'dhcp'
       
       #config device 'wan_dev'
       #       option name 'eth0.2'
       #       option macaddr '78:a3:51:2d:01:3b'
       
       #config interface 'wan6'
       #       option ifname 'eth0.2'
       #       option proto 'dhcpv6'

### 关闭 vlan2

       #config switch_vlan
       #       option device 'switch0'
       #       option vlan '2'
       #       option ports '4 6t'

### 为vlan1添加一个port

        option ports '0 1 2 3 6t'

改为
        option ports '0 1 2 3 4 6t'

这样子， 原来的wan口也能当lan口用了



### 关闭dhcp


       /etc/init.d/dnsmasq disable


### 关闭dhcpv6


       uci set dhcp.lan.dhcpv6=disabled
       uci set dhcp.lan.ra=disabled
       uci commit
       
       /etc/init.d/odhcpd disable





## 参考资料

https://openwrt.org/docs/guide-user/network/wifi/bridgedap 
