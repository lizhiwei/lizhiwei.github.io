<!--
author: lizhiwei
head: 
date: 2019-06-25
title: OpenVZ中配置Wireguard(Boringtun)
tags: IPv6
images: 
category: android
status: publish
summary: WireGuard是一个在Linux内核中实现的VPN软件，简单快速，加密强度高。本文介绍在Debian系统中的安装配置过程
-->


## OpenVZ类型的VPS 内核的版本是母机决定的，不能加载内核模块
   所以只有考虑 用户空间 的wiresguard实现， 基于性能考虑，选择了 boringtun
    


## 安装 boringtun

[编译boringtun](编译boringtun.html)


## 安装wireguard-tools

    # echo "deb http://deb.debian.org/debian/ unstable main" > /etc/apt/sources.list.d/unstable-wireguard.list
    # printf 'Package: *\nPin: release a=unstable\nPin-Priority: 90\n' > /etc/apt/preferences.d/limit-unstable
    # apt update
    # apt install wireguard-tools resolvconf

因为我们需要  wg 和 wg-quick 这两个工具



## 配置

   生成密钥对  创建配置文件 服务端配置数据转发 等操作 请 

   参考 [wireguard在Debian中的安装与配置](wireguard在Debian中的安装与配置.html)


  略有区别的是 服务端配置文件 wg0.conf

    echo "[Interface]
    PrivateKey = $(cat sprivatekey)
    Address = 10.0.0.1/24 
    PostUp   = iptables -t nat -A POSTROUTING -o venet0 -j MASQUERADE
    PostDown = iptables -t nat -D POSTROUTING -o venet0 -j MASQUERADE
    ListenPort = 6666
    MTU = 1420

    [Peer]
    PublicKey = $(cat cpublickey)
    AllowedIPs = 10.0.0.2/32"    | sed '/^#/d;/^\s*$/d' > wg0.conf  

主要是因为 openvz上iptables的写法有点不一样

    iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -j SNAT --to-source  8.8.8.8
    或
    iptables -t nat -A POSTROUTING -o venet0 -j MASQUERADE
    才可以

注意，OpenVZ7 7中（一般用 4.9.0 Linux内核), 需要使用

       iptables-legacy -t nat -A POSTROUTING -s 10.0.0.0/24 -o venet0    -j SNAT --to-source  1.2.3.4

也就是用 iptables-legacy 来代替  iptables

## 启动 boringtun

     # boringtun --disable-drop-privileges  --disable-multi-queue  --foreground  wg

KVM 或者 物理机器 可以用 multi-queue， 但是 OpenVZ不行
   
## 在OpenVZ上要进行修改


    @@ -132,7 +132,7 @@ fn main() {
             log_level,
             use_connected_socket: !matches.is_present("disable-connected-udp"),
             #[cfg(target_os = "linux")]
    -        use_multi_queue: !matches.is_present("disable-multi-queue"),
    +        use_multi_queue: false,   // !matches.is_present("disable-multi-queue"),
         };

         let mut device_handle = match DeviceHandle::new(&tun_name, config) {
    @@ -145,7 +145,7 @@ fn main() {
             }
         };
    
    -    if !matches.is_present("disable-drop-privileges") {
    +    if matches.is_present("disable-drop-privileges") {
             if let Err(e) = drop_privileges() {
                 eprintln!("Failed to drop privileges: {:?}", e);
                 sock1.send(&[0]).unwrap();

就是默认 带  --disable-drop-privileges  --disable-multi-queue 参数


然后重新编译 boringtun

## 启动服务

    # export PATH=/root/bin:/usr/sbin:/usr/bin:/sbin:/bin
    # WG_QUICK_USERSPACE_IMPLEMENTATION=boringtun  wg-quick up wg0

boringtun   放在 /root/bin 目录下





## 配置IPV6 tunnel地址

wg0.conf

    [Interface]
    PrivateKey = ...
    Address = 10.0.0.1/24,2001::1:2d98/128
    PreUp = ip -6 addr del 2001::1:2d98 dev venet0;ip -6 addr del 2001::1:7a5f dev venet0;ip -6 addr del 2001::1:1725 dev venet0
    PostUp   = iptables -t nat -A POSTROUTING -o venet0 -j MASQUERADE
    PostDown = iptables -t nat -D POSTROUTING -o venet0 -j MASQUERADE
    ListenPort = 6666
    MTU = 1420
    [Peer]
    PublicKey = ...
    AllowedIPs = 10.0.0.2/32,2001::1:7a5f/128

    

client0.conf

    [Interface]
    PrivateKey = ...
    Address = 10.0.0.2/24,2001::1:7a5f/128
    DNS = 8.8.8.8,2001:4860:4860::8888
    MTU = 1420
    [Peer]
    PublicKey = ...
    Endpoint = [2001::1:11b5]:6666
    AllowedIPs = 0.0.0.0/0, ::0/0
    PersistentKeepalive = 30
    


客户端运行： 

    wg-quick up client0

    [#] ip link add client0 type wireguard
    [#] wg setconf client0 /dev/fd/63
    [#] ip -4 address add 10.0.0.2/24 dev client0
    [#] ip -6 address add 2001::1:7a5f/128 dev client0
    [#] ip link set mtu 1420 up dev client0
    [#] resolvconf -a tun.client0 -m 0 -x
    
    [#] wg set client0 fwmark 51820
    
    [#] ip -6 route add ::/0 dev client0 table 51820
    [#] ip -6 rule add not fwmark 51820 table 51820
    [#] ip -6 rule add table main suppress_prefixlength 0
    
    [#] ip -4 route add 0.0.0.0/0 dev client0 table 51820
    [#] ip -4 rule add not fwmark 51820 table 51820
    [#] ip -4 rule add table main suppress_prefixlength 0
    

## 关于AllowedIPs

   AllowedIPs 这个词有误导性，不是说限制哪些IP可以连接。
   而是说哪些目的地址的IP需要路由到该节点的。
   0.0.0.0/0, ::0/0就是把所有的IPv4地址和IPv6地址都转发到服务端去
   我们 可以只指定  google ， twitter 等需要翻墙的IP范围

    并且，如果不在allowedips里指明 vpn gateway的地址的话

    AllowedIPs = 10.0.0.0/24

那么 10.0.0.1 也是ping不通的
