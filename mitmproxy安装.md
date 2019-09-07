<!--
author: lizhiwei
head: 
date: 2019-09-05
title: mitmproxy的安装
tags: https,sniffer,mitm
images: 
category: network
status: publish
summary: 介绍Linux中如何安装mitmproxy
-->


## 简单版

通过pip安装

    # apt-get python3-pip
　　$ pip3 install mitmproxy

会将依赖包　和　mitmproxy 安装到

~/.local/lib/python3.7/site-packages

二进制执行文件安装到

~/.local/bin

执行
~/.local/bin/mitmproxy
默认是normal proxy模式，监听在 8080 端口

## 开发版本

通过　virtualenv 　

    git clone https://github.com/mitmproxy/mitmproxy.git
    cd mitmproxy
    ./dev.sh 

执行

    . venv/bin/activate  
    mitmdump 





