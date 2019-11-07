<!--
author: lizhiwei
head: 
date: 2019-11-1
title: mitmproxy分析另外一台PC的流量
tags: mitmproxy
images: 
category: linux
status: publish
summary: mitmproxy
-->


## 参考资料

[mitmproxy透明模式分析reverse tethering手机流量](mitmprxy分析reverse_tethering的手机流量.html)

[Android 5.0之后反向usb网络共享](reverse_tethering.html) 

[Android中如何修改4G5G移动数据网络的DNS](Android手机上如何修改移动数据网络3G4G5G的DNS.html)

## 开启IP转发


    sysctl -w net.ipv4.ip_forward=1
    sysctl -w net.ipv6.conf.all.forwarding=1

检查

    cat /proc/sys/net/ipv4/ip_forward
    cat /proc/sys/net/ipv6/conf/all/forwarding 


## 设置iptable转发规则


    iptables -t nat -A PREROUTING  -p tcp --dport 80 -j REDIRECT --to-port 8080
    iptables -t nat -A PREROUTING  -p tcp --dport 443 -j REDIRECT --to-port 8080

如果要分析IP V6，也需要设置

    ip6tables -t nat -A PREROUTING  -p tcp --dport 80 -j REDIRECT --to-port 8080
    ip6tables -t nat -A PREROUTING  -p tcp --dport 443 -j REDIRECT --to-port 8080


## 启动mitmproxy

    mitmproxy --mode transparent --showhost


## 在被分析的机器上，将gateway设置为mitmproxy所在机器的IP

这样，所有的数据，都会发送到mitmproxy所在的机器



