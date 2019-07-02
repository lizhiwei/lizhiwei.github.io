<!--
author: lizhiwei
head: 
date: 2019-06-23
title: WireGuard在Debian中的安装与配置
tags: IPv6
images: 
category: android
status: publish
summary: WireGuard是一个在Linux内核中实现的VPN软件，简单快速，加密强度高。本文介绍在Debian系统中的安装配置过程
-->


## 环境
    因为wireguard官方版是实现在Linux kernel中的， 所以不能安装的OpenVZ VPS中
    只能装在KVM类型VPS,VirtualBox或物理机. 换句话说，只要能安装wireguard内核模块都行


## 安装 wireguard

    # echo "deb http://deb.debian.org/debian/ unstable main" > /etc/apt/sources.list.d/unstable-wireguard.list
    # printf 'Package: *\nPin: release a=unstable\nPin-Priority: 90\n' > /etc/apt/preferences.d/limit-unstable
    # apt update
    # apt install wireguard

## 生成密钥对

    cd /etc/wireguard
    wg genkey | tee sprivatekey | wg pubkey > spublickey
    wg genkey | tee cprivatekey | wg pubkey > cpublickey

## 创建配置文件

    echo "[Interface]
    PrivateKey = $(cat sprivatekey)
    Address = 10.0.0.1/24 
    PostUp   = iptables -t nat -A POSTROUTING -j MASQUERADE
    PostDown = iptables -t nat -D POSTROUTING -j MASQUERADE
    ListenPort = 6666
    MTU = 1420

    [Peer]
    PublicKey = $(cat cpublickey)
    AllowedIPs = 10.0.0.2/32"    | sed '/^#/d;/^\s*$/d' > wg0.conf  

   这是服务端配置文件


    echo "[Interface]
    PrivateKey = $(cat cprivatekey)
    Address = 10.0.0.2/24
    DNS = 8.8.8.8
    MTU = 1420

    [Peer]
    PublicKey = $(cat spublickey)
    Endpoint = $(curl -s whatismyip.akamai.com):6666
    AllowedIPs = 0.0.0.0/0, ::0/0
    PersistentKeepalive = 30" | sed '/^#/d;/^\s*$/d' > wg-client.conf

    这是客户端配置文件

## 服务端配置数据转发

    echo 1 > /proc/sys/net/ipv4/ip_forward
    echo 1  > /proc/sys/net/ipv6/conf/all/forwarding


或者修改  /etc/sysctl.conf

    net.ipv4.ip_forward=1
    net.ipv6.conf.all.forwarding=1


## 在服务器上启动服务端

    wg-quick up wg0

   注意: wg0 就是  /etc/wireguard 中的 配合文件名字

[#] ip link add wg0 type wireguard                                                                                                                      
[#] wg setconf wg0 /dev/fd/63                                                                                                                           
[#] ip -4 address add 10.0.0.1/24 dev wg0                                                                                                               
[#] ip link set mtu 1420 up dev wg0                                                                                                                     
[#] iptables -t nat -A POSTROUTING  -j MASQUERADE  


## 在PC客户端上同样方法安装 wireguard, 
   直接使用服务器上创建好的 wg-client.conf，放到 /etc/wireguard 目录
   wg-quick up wg-client

## 在移动设备上可以使用 Wireguard Android版 或者  Tunsafe Android版本
   直接导入 wg-client.conf
   或者使用qrencode 

    apt install qrencode
    qrencode -t ansiutf8 < wg-client.conf

会在控制台打印出二维码， 用手机版wireguard或者tunsafe扫描即可




