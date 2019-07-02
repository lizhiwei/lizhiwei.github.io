<!--
author: lizhiwei
head: 
date: 2019-06-23
title: OpenVZ中配置Wireguard-Go
tags: VPN
images: 
category: android
status: publish
summary: WireGuard是一个在Linux内核中实现的VPN软件，简单快速，加密强度高。本文介绍在Debian系统中的安装配置过程
-->


## OpenVZ类型的VPS 内核的版本是母机决定的，不能加载内核模块
   WireGuard-GO是一个完整的 用户空间实现
    


## 安装 WireGuard-Go

    cd /dev/shm
    wget https://dl.google.com/go/go1.12.6.linux-amd64.tar.gz
    tar xvzf go1.12.6.linux-amd64.tar.gz
    export PATH=$PATH:/dev/shm/go/bin
    mkdir gobuild
    cd gobuild
    git clone git://git.zx2c4.com/wireguard-go
    cd wireguard-go
    export GOPATH=/dev/shm/gobuild
    make wireguard-go
 在  /dev/shm/gobuild/wireguard-go 目录下生成可执行文件  wireguard-go
 
    


## 安装wireguard-tools

    # echo "deb http://deb.debian.org/debian/ unstable main" > /etc/apt/sources.list.d/unstable-wireguard.list
    # printf 'Package: *\nPin: release a=unstable\nPin-Priority: 90\n' > /etc/apt/preferences.d/limit-unstable
    # apt update
    # apt install wireguard-tools

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
    iptables -t nat -D POSTROUTING -o venet0 -j MASQUERADE
    才可以


## 启动 Go版本

申明使用 用户空间版本

    export WG_I_PREFER_BUGGY_USERSPACE_TO_POLISHED_KMOD=1

    wg-quick up wg0
   







