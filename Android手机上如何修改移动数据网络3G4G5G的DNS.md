<!--
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


