<!--
author: lizhiwei
head: 
date: 2019-09-10
title: mitmproxy透明模式分析reverse tethering手机流量
tags: https,sniffer,mitm
images: 
category: network
status: publish
summary: mitmproxy透明模式分析手机流量
-->


## 反检测

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


    iptables -t nat -A PREROUTING  -p tcp --dport 80 -j REDIRECT --to-port 8080
    iptables -t nat -A PREROUTING -i enp0s20f0u5 -p tcp --dport 443 -j REDIRECT --to-port 8080

（　-i enp0s20f0u5　选项可以不需要　enp0s20f0u5　为 usb网卡）

### 关闭　masquerade (可选，关闭了DNS可能用不了)

    iptables -t nat -D POSTROUTING -j MASQUERADE

如果真要关闭masquerade,那么在PC上运行一个　dnsmasq, 并将　[修改dns](Android%E6%89%8B%E6%9C%BA%E4%B8%8A%E5%A6%82%E4%BD%95%E4%BF%AE%E6%94%B9%E7%A7%BB%E5%8A%A8%E6%95%B0%E6%8D%AE%E7%BD%91%E7%BB%9C3G4G5G%E7%9A%84DNS.html) 中的　iptable指令改为


    iptables -t nat -A PREROUTING -p udp -d 218.104.111.114 --dport 53 -j DNAT --to 127.0.0.1:53
    iptables -t nat -A PREROUTING -p udp -d 218.106.127.114 --dport 53 -j DNAT --to 127.0.0.1:53
    iptables -t nat -A PREROUTING -p udp -d 218.104.111.122 --dport 53 -j DNAT --to 127.0.0.1:53 



### 透明模式启动mitmproxy

     ~/.local/bin/mitmproxy  -T --host
     或者
     ~/.local/bin/mitmproxy --mode transparent --showhost
  

