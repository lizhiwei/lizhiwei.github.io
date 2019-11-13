<!--
author: lizhiwei
head: 
date: 2019-05-03
title: OpenWRT中配置IPV6
tags: IPv6
images: 
category: network
status: publish
summary: 在OpenWRT中配置如何从中国移动光纤宽带中获取IPv6地址
-->


## 准备材料
  1. 智博通 WG3526 路由器 (MT7621A 16M ROM,  512M RAM)
  2. 中国移动光纤入户 (使用Prefix Delegation前缀委托模式)
    或者 中国联通FTTH
  
## 刷机
  [OpenWRT 18.06 for ZBT WG3526](http://downloads.openwrt.org/releases/18.06.2/targets/ramips/mt7621/openwrt-18.06.2-ramips-mt7621-zbt-wg3526-16M-squashfs-sysupgrade.bin)

持续捅住reset按钮5秒，同时通电开机， 用浏览器打开 http://192.168.1.1/  上传sysupgrade.bin

或者 自行  [编译OpenWRT](build_openwrt.html)



## 地址分配介绍

在IPv6网络中，PPPoE接入方式的逐层配置结构没有变化，只是PPPoE协议族中， 除了IPCP协议外，还增加了IPv6CP协议。

在PPPoE链接建立后，只完成接口ID的协商，有两种方式得到一个完整的全球单播IPv6地址:

1. 通过ND协议获得IPv6前缀和PPPoE协商的IPv6接口ID组合成全球单播IPv6地址，

2. PPPoE认证成功后，通过DHCPv6协议下发IPv6前缀或全球单播IPv6地址。

中国联通和中国移动都差不多
地址获取用 SLAAC
前缀获取是 DHCPv6


具体为：
- IPv6CP将用于链路本地地址分配（LLA）， 也就是pppoe-wan中的inet6地址
- DHCPv6的前缀委派（IA-PD）用于获取局域网地址前缀,  也就是 br-lan 中  ::1/60 前缀
- 无状态DHCPv6用于获取其他配置参数， 比如隐私扩展地址



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
        
        
        
关闭IPV6防火墙，否则本机的一些端口无法暴露到公网

/etc/config/network

关闭global prefix

    #config globals 'globals'
    #   option ula_prefix 'fdca:59d7:4008::/48'


配置pppoe(加自动ipv6)

    config interface 'wan'
        option ifname 'eth0.2'
        option proto 'pppoe'
        option username '...'
        option password '...'
        option ipv6 'auto'

关闭wan6

    #config interface 'wan6'
    #       option ifname 'eth0.2'
    #       option proto 'dhcpv6
    
因为上面lan的ipv6设置了auto， 所以ipv6 从 ppp-wan口 获取 ，而不是 eth0.2

## 显示路由表


    # ip -6 route show
   

....




## ifstatus查看PD分配

    ifstatus lan
    
    查看 ipv6-prefix-assignment
    
    

