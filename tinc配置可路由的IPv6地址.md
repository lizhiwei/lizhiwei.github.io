<!--
author: lizhiwei
head: 
date: 2019-05-06
title: tinc vpn配置可路由的native IPV6
tags: IPv6
images: 
category: network
status: publish
summary: 配置tinc VPN, 给予客户端结点可路由native ipv6地址，而不是nat转换的地址
-->


[vultr](https://www.vultr.com/?ref=8050818-4F)安装系统时选择支持IPv6, 分配了一个 2001:1234:abcd:56/64的前缀

其他KVM类型的VPS，提交support ticket,申请若干个IP V6地址，工作人员会手工分配地址，在SolusVM控制面板上看到绑定了几个地址，
但其实是一个 2607:1234:abcd:56/64的地址块，只不过 ::1 和 ::2已经被网关占用，不可分配给虚拟机



## 网络配置方案


国外路由节点（vultr或者其他服务器)
ipv6   2001:1234:abcd:56::10/120


国内节点(自己的Linux PC)
ipv6   2001:1234:abcd:56::f/120

因为 2001:1234:abcd:56::1 和  2001:1234:abcd:56::2 在某些小型vps供应商提供的的KVM虚拟机中，已经被网关占用，为了统一起见
我们没有使用 ::1 和  ::2 地址分配给国内节点


## 服务器软件配置

### 安装Debian GNU/Linux buster

vultr 直接是用 buster的netinst ISO镜像安装的



### 网卡配置


/etc/network/interfaces

alpharacks可用dhcp

    auto eth0
    iface eth0 inet dhcp


或static

    iface eth0 inet static
    address 204.111.222.33
    gateway 204.152.220.129
    netmask 255.255.255.128
    dns-nameservers 8.8.8.8 8.8.4.4

    up ip -6 addr add 2001:1234:abcd:56::9:9/64 dev eth0    
    up ip -6 route add default via fe80::2d0:2ff:fed8:5c00 dev eth0

在vultr就简单了




## 服务端配置

### ip转发
    echo 1  > /proc/sys/net/ipv6/conf/all/forwarding
    或者
    sysctl -w net.ipv6.conf.all.forwarding=1

### ndp代理
    echo 1  > /proc/sys/net/ipv6/conf/all/proxy_ndp
    或者
    sysctl -w net.ipv6.conf.all.proxy_ndp=1

如果修改 /etc/sysctl.conf, 需要打开

    net.ipv4.ip_forward=1
    net.ipv6.conf.all.forwarding=1
    net.ipv6.conf.all.proxy_ndp=1
    net.ipv6.conf.eth0.accept_ra=2 （因为开启了forwarding, 所以accept_ra要=2,否则会忽略路由器通告)


### /etc/tinc/lizhiwei/tinc.conf
    Name=vultr
    Mode=switch
    Interface=tun0
    PrivateKeyFile=/etc/tinc/lizhiwei/rsa_key.priv

注意：模式是switch, 而不是router
   也就是将 /dev/net/tun 作为一个tap设备

   原因是ipv6 要支持 multicast， ndp协议

### /etc/tinc/lizhiwei/tinc-up

    #!/bin/sh

    OLDIF=`ip -6 route show  proto ra | grep ^default | cut -d ' ' -f 4-5`
    
    ip -6 link set $INTERFACE up mtu 1450  txqueuelen 1000
    
    ip -6 addr add 2001:1234:abcd:56::10/120 dev $INTERFACE
    ip -6 neigh add proxy 2001:19f0:ac01:3bc::2   $OLDIF

### 服务端和客户端的公钥

/etc/tinc/lizhiwei/hosts/vultr

    -----BEGIN RSA PUBLIC KEY-----
    ...
    -----END RSA PUBLIC KEY-----
    
    Address=2001:1234:abcd:56:5400:2ff:fe01:ef83
    Port=9527

/etc/tinc/lizhiwei/hosts/ips4k

    -----BEGIN RSA PUBLIC KEY-----
    ...
    -----END RSA PUBLIC KEY-----
    Port=9528









## 客户端配置
### /etc/tinc/lizhiwei/tinc.conf

    Name=ips4k
    Mode=switch
    Interface=tun0
    PrivateKeyFile=/etc/tinc/lizhiwei/rsa_key.priv
    ConnectTo=vultr

注意： 客户端有 ConnectTo 配置选项



### /etc/tinc/lizhiwei/tinc-up 

    ip -6 link set $INTERFACE up mtu 1450
    ip -6 addr add 2001:1234:abcd:56::f/120 dev $INTERFACE

tinc起来时，设置mtu, 设置ip地址


### 客户端服务器公钥配置

/etc/tinc/lizhiwei/hosts/ips4k

    -----BEGIN RSA PUBLIC KEY-----
    ...
    -----END RSA PUBLIC KEY-----


/etc/tinc/lizhiwei/hosts/vultr

    ...



### /etc/tinc/lizhiwei/host-up
    #!/bin/sh

    VPN_GATEWAY=2001:19f0:ac01:3bc::10
    ORIGINAL_GATEWAY=`ip -6 route show proto ra | grep ^default | cut -d ' ' -f 4-5`
  
    #ip -6 route add $REMOTEADDRESS $ORIGINAL_GATEWAY

    #ip -6 route add default via $VPN_GATEWAY 
    #ip -6 route del default $ORIGINAL_GATEWAY

    

主机起来时，
  删除物理接口的局域网路由，防止接受ra通告
  给远程ipv6地址设置路由， 走原始的物理出口
  默认路由设置为vpn网关， 
  删除原始路由


## 运行

国外节点 

    tincd -n lizhiwei
    iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -j SNAT --to-source  1.2.3.4


国内

    tincd -n lizhiwei
    gfw (这个脚本添加了 google 等黑名单，内容可参考下面)





## 注意事项
   本地默认物理接口的路由不好删除，ra通告总是会发出来， 客户端接收到路由公告之后，会添加物理接口的默认路由

   暂时，只能将 google 等被屏蔽网站，走vpn路由， 其他网站，走默认的物理路由


### 手动添加名单

    #github.io
    ip route add 185.199.0.0/16 via 10.0.0.1

    #twitter.com
    ip route add 104.244.42.0/24 via 10.0.0.1

    # dns
    ip -6 route add  2001:4860:4860::/48 via 2001:1234:abcd:56::10

    # google
    ip -6 route add  2607:f8b0:4000::/44 via 2001:1234:abcd:56::10

    # wiki
    ip -6 route add  2620:0:863:ed1a::/64 via 2001:1234:abcd:56::10

    # whatismyv6
    ip -6 route add  2607:f0d0:3802:84::/64 via 2001:1234:abcd:56::10



## 客户端VPN端地址变成全局可路由

    2001:1234:abcd:56::f

    可以被其他配置好IPV6的主机，直接访问





## 关于vultr 和 linode 的一点说明


Linode给你/64网段的方法，是在他们的路由表里面，把所有到这个网段的包全都路由到你的主机上去。
而Vultr的IPv6，上来就给你的是一个/64的网段，你在Vultr的主机本身也只是监听Router Advertisement来自动配置IPv6地址而已。
Vultr那边，不会帮你把到那个地址的所有包全都自动路由到你的主机，而是你得自己去响应Vultr的路由器的IPv6 Neighbor Discovery这种东西，让Vultr知道你的存在，他的路由器才会理你。

所以， vultr 的 vps 需要开启  proxy_ndp 

 而 linode 不需要





