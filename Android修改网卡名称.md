<!--
author: lizhiwei
head: 
date: 2019-10-19
title: Android修改网络接口名称
tags: Android
images: 
category: Android
status: publish
summary: 修改模拟器中修改网卡名称
-->


## 在Android Emulator中，网卡名称可能是
    eth1

##  修改

    ip link set eth0 down
    ip link set eth0 name rmnet0
    ip link set rmnet0 up

## 检查网络接口名称

    #  ls /sys/class/net
    hwsim0 rmnet0 lo sit0 wlan0 wlan1 


rename　a　linux　network　interface　without　udev


