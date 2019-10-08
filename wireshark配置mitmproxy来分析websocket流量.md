<!--
author: lizhiwei
head: 
date: 2019-10-08
title: wireshark配置mitmproxy来分析websocket流量
tags: mitmproxy
images: 
category: network
status: publish
summary: 用mitmproxy得到相应的key,存入log, wireshark利用这些key俩解密websocket流量
-->

按照　[mitmproxy分析手机流量](mitmprxy分析reverse_tethering的手机流量.html) 准备好环境

将运行mitmproxy的命令改为

    SSLKEYLOGFILE="/dev/shm/sslkeylogfile.txt"  ~/.local/bin/mitmproxy --mode transparent --showhost

这样，就将相应的key写入到　sslkeylogfile.txt中

也可以用export命令，如:

    export　SSLKEYLOGFILE="/dev/shm/sslkeylogfile.txt"
    ~/.local/bin/mitmproxy --mode transparent --showhost

如果　同Firefox或者Chrome冲突，设置　MITMPROXY_SSLKEYLOGFILE　环境变量也是可以的


运行mitmproxy的时候，同时启动　wireshark

然后，在　Edit -> Preferences -> Protocols -> SSL -> (Pre)-Master-Secret log filename 设置里
设置　log文件，　即可对流量进行解密







