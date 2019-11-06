<!--
author: lizhiwei
head: 
date: 2019-10-19
title: Android NDC命令
tags: Android
images: 
category: Android
status: publish
summary: ndroid NDC命令
-->


## ndc用来设置网卡或者修改路由


### ndc ipfwd status
211 0 Forwarding disabled

### ndc ipfwd status
211 0 Forwarding enabled

### ndc interface list

### ndc interface getcfg wlan0
213 0 00:ec:0a:70:85:92 192.168.1.124 24 up broadcast running multicast

### ndc ipfwd add tun0 wlan0

### ndc nat enable rndis0 tun0


### ndc tether status
210 0 Tethering services started

### ndc tether interface list
111 0 rndis0
200 0 Tether operation succeeded

### ndc tether dns list
115 0 100
112 0 fd3b:fbcb:a1a::1
112 0 192.168.1.1
200 0 Tether operation succeeded


###  ndc netd network route add



### ndc nat enable tun0 wlan0 1  



如何将VPN共享给wifi热点


