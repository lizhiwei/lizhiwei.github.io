app<!--
author: lizhiwei
head: 
date: 2019-09-09
title: Android中如何修改4G5G移动数据网络的DNS
tags: https,sniffer,mitm
images: 
category: network
status: publish
summary: Android早期版本可以改属性
-->


## Android 2.x时代

    setprop net.dns1  8.8.8.8
    setprop net.dns1  8.8.4.4

## Android 4.x 时代

    ndc resolver setifdns rmnet1 8.8.8.8 8.8.4.4

## Android 6.0 - Android 9.0时代


    ndc resolver setnetdns  rmnet_data3 114.114.114.114 114.114.115.115 

返回正确：200 0 Resolver command succeeded


## 用iptables来拦截

　usb reverse tethering, 在PC上设置

iptables -t nat -A PREROUTING -p udp  -d  218.104.111.122  --dport 53 -j DNAT --to 114.114.114.114:53

iptables -t nat -A PREROUTING -p udp  -d  218.104.111.122  --dport 53 -j DNAT --to 114.114.114.114:53


在Android手机上，自己拦截

iptables -t nat -A OUTPUT -p udp -d 218.104.111.122  --dport ５３ -j DNAT --to 114.114.114.114:53

或者无差别的拦截

 iptables　-t nat -A OUTPUT -p udp --dport 53 -j DNAT　--to 114.114.114.114:53
 iptables　-t nat -A OUTPUT -p tcp --dport 53 -j DNAT　--to 114.114.114.114:53



 


