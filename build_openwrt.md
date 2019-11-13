<!--
author: lizhiwei
head: 
date: 2019-11-11
title: 编译OpenWRT
tags: IPv6
images: 
category: network
status: publish
summary: 编译OpenWRT
-->


## 准备编译环境

       apt install build-essential libncurses-dev gawk git  gettext zlib1g-dev  unzip 
  
## 准备源代码

       git clone https://git.openwrt.org/openwrt/openwrt.git
       

如果要切换到稳定版，进入源代码目录后执行

       git checkout openwrt-18.06

##  更新并安装包

       ./scripts/feeds update -a
       ./scripts/feeds install -a


## 配置编译参数

       make menuconfig



## 参考资料


https://openwrt.org/docs/guide-developer/build-system/install-buildsystem




    

