<!--
author: lizhiwei
head: 
date: 2019-07-03
title: 用ping检测PMTU
tags: IPv6
images: 
category: IPV6
status: publish
summary: IPV6环境下的PMTU
-->


## 背景介绍
最大传输单位是您可以在接口上发送和接收的最大网络层字节数。 “网络层”意味着计数不包括任何链路层成帧，例如以太网报头或SDH段开销，但确实包括网络层报头，例如IP或IPv6数据包报头。

以太网MTU为1500.实际上，所有以太网接口都可以发送稍多的数据，以允许链路层扩展标头，如VLAN或MPLS标签。

许多千兆位以太网接口可以发送更大的数据包：通常但不总是超过9000字节。这些“巨型帧”（jumbo frames)没有被IEEE标准化，因为在他们看来，以太网应该与所有其他以太网互操作。学术和研究网络在Internet2联合技术会议上讨论了巨型帧，并宣称他们的网络将通过9000字节的巨型帧，这一决定被APAN和TERENA采用，似乎也成为商业惯例。


那次会议做出了另一个决定：巨型帧的目的是提供从主机接口到主机接口的9000字节路径。这意味着使用巨型帧的工程实践与使用标准帧的工程实践略有不同 - 在运行隧道时，您设计网络以向通过隧道的流量提供9000字节。


那么如何判断你是否有一个9000字节的干净路径？使用ICMP Echo Request发送巨型帧，设置IP的Do Not Fragment位。如果您收到ICMP Echo Reply，则该路径不能将您的巨型帧分段。

这应该很简单。不幸的是，ping程序的“大小”参数表示不同的意义。网络工程师希望大小为整个网络层的大小，但ping程序 实现“大小”作为ICMP Echo Request有效负载中的字节数更为简单。没有选项的IPv4报头是20字节，ICMP报头是8字节，没有选项的IPv6报头是40字节，ICMP6报头是4字节。


## 用ping来检测

### 计算IPV4 MTU
    Linux ping, IPv4
    size = mtu − icmpv4_header − ipv4_header
    size = mtu − 8 − 20
    size = mtu − 28

### 检查1500MTU

在 郑州景安网络的服务器上检测
    ping -n  -4 -c 1 -M do -s 1472 zhiwei.li
    PING zhiwei.li (198.23.128.123) 1472(1500) bytes of data.
    1480 bytes from 198.23.128.123: icmp_seq=1 ttl=48 time=181 ms

在 中国移动PPPoE光纤宽带中检测

    ping  -4 -c 1 -M do -s 1472 zhiwei.li
    PING zhiwei.li (198.23.128.123) 1472(1500) bytes of data.
    From 192.168.1.1 (192.168.1.1) icmp_seq=1 Frag needed and DF set (mtu = 1492)

这也容易解释：因为IP数据封装在PPPOE中， PPP头部占据了8个字节

## 计算IPV6 MTU

Linux ping6, IPv6
size = mtu − icmpv6_echo_header − icmpv6_header − ipv6_header
size = mtu − 4 − 4 − 40
size = mtu − 48

### 检测1500 MTU

    ping6 -n -c 1 -M do -s 1452 ipv6.zhiwei.li
    PING ipv6.zhiwei.li(2001:f000:f000:1ee1::9) 1452 data bytes
    1460 bytes from 2001:f000:f000:1ee1::9: icmp_seq=1 ttl=241 time=194 ms

因为 zhiwei.li 的 IPV6是 He.net 提供的， 用IPV4封装的，所以，又要减去20个字节头部开销

    ping6 -n  -c 1 -M do -s 1432 zhiwei.li
    PING zhiwei.li(2001:470:1f04:27::2) 1432 data bytes
    1440 bytes from 2001:470:1f04:27::2: icmp_seq=1 ttl=52 time=183 ms
    
改成 1482 就会报错
    ping6 -n  -c 1 -M do -s 1434 zhiwei.li
    PING zhiwei.li(2001:470:1f04:27::2) 1434 data bytes
    From 2001:470:0:45::2: icmp_seq=1 Packet too big: mtu=1480

在中国移动PPPoE宽带中检测，还是要减去8个头部

    ping6 -n  -c 1 -M do -s 1444 ipv6.zhiwei.li

用wireguard封装后，继续减少72个字节
  ping6 -n  -c 1 -M do -s 1372 

## 后记


IPv4和IPv6碎片策略之间存在细微差别。 IPv4路由器在需要时对网络中的流量进行分段，然后接收主机重新组装这些分段。这通常很有效，但存在许多潜在问题。由于这些问题，IETF开发了用于更高层协议（如TCP）的方法，以确定路径上的最小MTU并发送适当大小的数据报以避免碎片。 IPv6设计者假设存在此路径MTU发现，因此在IPv6中，碎片不再发生在网络中，而是仅发生在主机上 - 然后仅在特殊情况下才会出现这种情况。

设计师们也做了另一个假设。由于现代网络使用1500字节或更大的MTU，因此它们将IPv6 MTU最小值576字节提高到1500字节。现在，当使用隧道时，必须减少隧道MTU，以便外部数据报不大于链路MTU（通常为1500字节）。因此，IPv6主机将接受低至1280字节的路径MTU，但不一定更小。


明智的一句话：在今天的IPv6互联网中，有很多隧道。这将改变，但由于IP / IP VPN，它们永远不会消失。不要过滤掉ICMPv6数据包太大或目的地无法到达的消息，虽然您可能对它们进行速率限制，但不要将路由器配置为不发送它们，也不要让主机不接受它们。它们使网络能够为问题的传输提供信息，从而解决问题。



问：何为路径 MTU 探测（PMTUD）？
PMTUD 是一种能告诉你的电脑向某一目标发送的数据包的最大尺寸的机制。原理如下：你的电脑会发送一个很大的数据包，并标注“不要分段”，当通过一个无法接受该尺寸的路由器时，路由器就会回复说“太大了！试试这个大小吧。”

问：这和 IPv6 有什么关系？
在 IPv6 中，所有数据包都是“不要分段”的。发送大小合适的数据包的责任不在路由器上了，因为这对路由器来说是很大的负担。IPv6 规范规定了 PMTUD 功能来解决这一问题。

问：防火墙需要放行什么？
IPv6 防火墙必须放行 ICMPv6 第 2 类消息（“数据包过大”）以与公网兼容。如果你正为你的网站、企业或其他组织搭建 IPv6 防火墙，即使禁止了其他类型的 ICMP，也请放行这类 ICMPv6 消息。

问：还有什么会导致 PMTUD 错误？
多重隧道。其中一个可能是你自己的，而另外一个你可能看不到。运营商经常使用隧道来简化或隐藏他们的网络结构，或者方便数据传输。然而，多用一个隧道就得多加一个包头，使数据包变大，可问题在于路由器对数据包的大小是有限制的。






    
