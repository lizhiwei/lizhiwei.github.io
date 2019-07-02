<!--
author: lizhiwei
head: 
date: 2019-05-04
title: 邻居发现协议NDP
tags: IPv6
images: 
category: network
status: publish
summary: IPv6中的NDP
-->


## 实例分析

1.邻居征集 Neighbor Solicitation (135)

    Mac Dst:  IPv6mcast_ff:55:45:e9
    IP  Dst:  ff02::1:ff55:45e9


2.路由器征集  Router Solicitation (133)

    Mac Dst:  IPv6mcast_02
    IP Dst:   ff02::2


3.路由通告	Router Advertisement

   Flags: Default Router Preference: Medium
  
  ICMPv6 选项 ：

    Prefix information:  2409:8a4c:ca2c:5d80::
    Route Information : Medium 2409:8a4c:ca2c:5d80::/60
    Recursive DNS Server 2409:8a4c:ca2c:5d80::1

4.DHCPv6 报告自己的IPv6地址

    Message type: Confirm (4)
    请求选项：Requested Option code: DNS recursive name server (23)
            Requested Option code: Domain Search List (24)
            Requested Option code: DNS recursive name server (23)
    主机名： Option: Fully Qualified Domain Name (39)
    非临时地址： Option: Identity Association for Non-temporary Address (3)

5.服务器回复

    Message type: Reply (7)
    确认地址，并回复DNS
    Option: DNS recursive name server (23)
         DNS server address: 2409:8a4c:ca2c:5d80::1

其实在RA里已经告知了DNS服务器地址，就是路由器br-lan接口的地址


6.DHCPv6 Message type: Solicit (1)

7.DHCPv6 Message type: Advertise (2)

Solicit/Advertise 干的事情 跟 Confirm/Reply 差不多，不知道什么还要有这么一趟流程



4.根据路由器告示的前缀，再次发出 邻居征集 Neighbor Solicitation (135)

只是Target Address变成了 2409:8a4c:ca2c:5d80:428d:5cff:fe55:45e9




