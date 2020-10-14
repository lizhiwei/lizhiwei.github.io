<!--
author: lizhiwei
head: 
date: 2020-02-12
title: OpenWRT生成etc目录下的配置
tags: IPv6
images: 
category: network
status: publish
summary: OpenWRT生成etc目录下的配置
-->


## LAN口和WAN口分配

target/linux/ramips/dts/mt7620a_zbtlink_zbt-we826.dtsi 中
    
        &ethernet {
               mtd-mac-address = <&factory 0x4>;
       -       mediatek,portmap = "wllll";
       +       mediatek,portmap = "llllw";
        };
       

在 target/linux/generic/files/include/linux/switch.h 定义了 

       struct switch_portmap {


target/linux/generic/files/drivers/net/phy/swconfig.c

       of_switch_load_portmap




## 配置生成脚本

       package/base-files/files/bin/config_generate

## 默认配置

       package/network/config/firewall/files/firewall.config 对应  /etc/config/firewall

