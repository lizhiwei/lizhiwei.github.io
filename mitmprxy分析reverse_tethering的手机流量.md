<!--
author: lizhiwei
head: 
date: 2019-09-09
title: mitmproxy透明模式分析reverse tethering手机流量
tags: https,sniffer,mitm
images: 
category: network
status: publish
summary: mitmproxy透明模式分析手机流量
-->


## 好处

### 有些App会检查是否使用移动数据网络

如果不是，行为可能不同

### 有些应用会检查wifi是否开启了proxy

如果是，可能报告自己被sniffer了

### 有些应该可能会检查VPN是否开启
如果是，可能报告自己被sniffer了



### Magisk能很好地隐藏root

    magiskhide

## 步骤

### 参照　[reverse tethering](reverse_tethering.html) 
配置　移动数据网络　和　反向usb网络共享

### 在PC上设置好端口重定向


    iptables -t nat -A PREROUTING -i enp0s20f0u5 -p tcp --dport 80 -j REDIRECT --to-port 8080
    iptables -t nat -A PREROUTING -i enp0s20f0u5 -p tcp --dport 443 -j REDIRECT --to-port 8080

其中　enp0s20f0u5　为 usb网卡

### 关闭　masquerade (可选，关闭了DNS可能用不了)

    iptables -t nat -D POSTROUTING -j MASQUERADE



### 透明模式启动mitmproxy

     ~/.local/bin/mitmproxy  -T --host
     或者
     ~/.local/bin/mitmproxy --mode transparent --showhost
  

