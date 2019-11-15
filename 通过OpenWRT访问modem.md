<!--
author: lizhiwei
head: 
date: 2019-11-15
title: 通过OpenWRT直接访问modem的管理界面
tags: OpenWRT
images: 
category: network
status: publish
summary: 通过OpenWRT直接访问modem的管理界面
-->



第一种方法是
## 配置network和firewall

/etc/config/network 增加一个接口

       config interface 'modem'
              	option ifname 'eth0.2'
              	option proto 'static'
              	option ipaddr '192.168.1.99'
              	option netmask '255.255.255.0'
       
       
/etc/config/firewall


       config zone
              	option name		wan
              	list   network		'wan'
               #list   network	'wan6'
              	option input		REJECT
              	option output		ACCEPT
              	option forward		REJECT
              	option masq		1
              	option masq_dest        '!modem'
              	option mtu_fix		1


添加一个选项 masq_dest , 值为  !modem


或者修改firewall配置


       uci set firewall.@zone[1].network='wan modem'
       uci commit firewall
       fw reload






另外一种方法，就是
## 命令行配置


       ifconfig  eth0.2:1  add 192.168.1.99 
       
       iptables -t nat -I postrouting_rule -s 192.168.0.0/24 -d 192.168.1.1 -j SNAT --to 192.168.1.99
       iptables -I zone_lan_forward -s 192.168.0.0/24 -d 192.168.1.1 -j ACCEPT
       

上面的命令适用于

OpenWRT 局域网段  192.168.0.*

modem的IP 192.168.1.1



