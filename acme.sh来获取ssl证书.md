<!--
author: lizhiwei
head: 
date: 2019-11-12
title: 用acme.sh来获取Let's Encrypt免费SSL证书
tags: IPv6
images: 
category: network
status: publish
summary: 我认为是最简单方便同时又最完善的
-->


## 安装

       git clone https://github.com/Neilpang/acme.sh.git
       cd acme.sh
       ./acme.sh --install
  
## 以dns模式手动运行

       ~/.acme.sh/acme.sh --issue --dns     -d  zhiwei.li -d *.zhiwei.li  --yes-I-know-dns-manual-mode-enough-go-ahead-please  

按照提示，设置两个txt记录

acme-challenge.zhiwei.li

然后renew

       ~/.acme.sh/acme.sh --renew   --dns -d  zhiwei.li -d *.zhiwei.li   --yes-I-know-dns-manual-mode-enough-go-ahead-please  
       



##  或者以DNS API运行

它支持非常多的dns API, 比如name.com就有


       export Namecom_Username="testuser"
       export Namecom_Token="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
       
       
       acme.sh --issue --dns dns_namecom -d zhiwei.li -d *.zhiwei.li

或者 namesilo.com 

       export Namesilo_Key="xxxxxxxxxxxxxxxxxxxxxxxx"
       acme.sh --issue --dns dns_namesilo --dnssleep 900 -d zhiwei.li -d *.zhiwei.li

或者internet.bs

       export INTERNETBS_API_KEY="..."
       export INTERNETBS_API_PASSWORD="..."

       acme.sh --issue --dns dns_internetbs -d zhiwei.li -d *.zhiwei.li



## he.net没有API，可以模拟登录

       export HE_Username="yourusername"
       export HE_Password="password"
       
       acme.sh --issue --dns dns_he -d zhiwei.li -d *.zhiwei.li
       



## 参考资料


https://github.com/Neilpang/acme.sh/wiki/How-to-install

https://github.com/Neilpang/acme.sh/wiki/DNS-manual-mode

https://github.com/Neilpang/acme.sh/wiki/dnsapi
        




    

