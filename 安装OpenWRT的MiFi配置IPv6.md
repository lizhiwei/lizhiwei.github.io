<!--
author: lizhiwei
head: 
date: 2019-05-03
title: 给MiFI配置IPV6
tags: IPv6
images: 
category: network
status: publish
summary: 在OpenWRT中配置如何从中国移动光纤宽带中获取IPv6地址
-->


## 准备材料
  1. 智博通 CPE103 路由器 (MT7620A 16M ROM,  128M RAM)
  2. 中国联通/中国电信/中国移动　SIM卡一张
  

## 配置

/etc/config/dhcp

    config dhcp 'lan'
         ...
        option dhcpv6 'server'
        option ra 'server'

dchpv6和ra(Router Advertisement)配置,保持不变,均为'server'

/etc/config/firewall

    config defaults
        ...
        # Uncomment this line to disable ipv6 rules
        option disable_ipv6     1
....

    config zone                           
            option name             wan   
            list   network          'lte' 
            option input            REJECT
            option output           ACCEPT
            option forward          REJECT
            option masq             1     
            option mtu_fix          1     
                                          
    config forwarding                     
            option src              lan
            option dest             wan

        
关闭IPV6防火墙，否则本机的一些端口无法暴露到公网

/etc/config/network

关闭global prefix

    #config globals 'globals'
    #   option ula_prefix 'fdca:59d7:4008::/48'


    config interface 'lte'           
            option pdptype 'ipv4v6'  
            option device '/dev/cdc-wdm0'
            option proto 'qmi'           
            option dhcpv6 '1'





