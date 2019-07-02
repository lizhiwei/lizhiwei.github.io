<!--
author: lizhiwei
head: 
date: 2019-06-25
title: 给单IPV4地址的OpenVZ VPS配置TunnelBrokerIPV6
tags: VPN
images: 
category: Linux
status: publish
summary: 有些小型的OpenVZ VPS供应商不提供IPV6地址，可以用he.net tunnelbroker来提供一个/48的IPV6地址
-->


## OpenVZ类型的VPS 内核 不支持sit模块
   只好用一个用户空间的实现


## 编译  TunnelBroker Tun

    git clone https://github.com/acgrid/tb-tun
    cd tb-tun
    make

不过 [acgrid](https://github.com/acgrid) 本来提供一个编译好的 tb_userspace 可执行文件, 可以直接用



## 配置

    setsid ./tb_userspace tb HeServerIPv4Address MyClientIPv4Address sit > /dev/null
    sleep 6
    ip link set tb up                          # ifconfig tb up
    ip addr add MyClientIPv6Address dev tb     # ifconfig tb inet6 add MyClientIPv6Address
    ip link set dev tb mtu 1480                # ifconfig tb mtu 1480
    ip route add ::/0 dev tb                   # route -A inet6 add ::/0 dev tb
    ip -6 route del default dev venet0

## 将上述命令放在 /etc/rc.local

   即可开机自动配置

如果是systemd的系统，需要启用

    systemctl enable rc-local


## Tunnel Broker 申请地址


[HE.net IPv6 Tunnel Broker Registration](https://tunnelbroker.net/register.php)


