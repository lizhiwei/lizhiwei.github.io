<!--
author: lizhiwei
head: 
date: 2019-11-13
title: 安装gfw-trojan来翻越gfw
tags: IPv6
images: 
category: network
status: publish
summary: 最厉害的番羽土啬方案，隐藏在正常的https流量中
-->


## 安装

      apt install trojan
      
  
## 或者从源代码安装

依赖

CMake >= 3.7.2

Boost >= 1.66.0

OpenSSL >= 1.0.2

libmysqlclient


       apt -y install build-essential cmake libboost-system-dev libboost-program-options-dev libssl-dev default-libmysqlclient-dev
       
获取源代码

       git clone https://github.com/trojan-gfw/trojan.git
       cd trojan/
     
编译：
       mkdir build
       cd build/
       cmake ..
       make



##  配置服务端

       ...
       "local_addr": "::",
       ...
        "password": [
               "MyPassword1",
               "MyPassword2"
           ],

       ...
     "ssl": {
         "cert": "/home/zhiwei/ssl/zhiwei.cer",
         "key": "/home/zhiwei/ssl/zhiwei.key",
       ...

其他选项保持默认就好

## 配置客户端


       "run_type": "client",
       ...
       "local_port": 1080,
       "remote_addr": "1.1.1.1",
       "remote_port": 443,
       "password": [
              "password1",
              "password2"
       ],

ssl 改成

    "ssl": {
        "verify": true,
        "verify_hostname": true,
        "cert": "/etc/trojan/zhiwei.cer",
        "cipher": "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-RSA-AES128-SHA:ECDHE-RSA-AES256-SHA:RSA-AES128-GCM-SHA256:RSA-AES256-GCM-SHA384:RSA-AES128-SHA:RSA-AES256-SHA:RSA-3DES-EDE-SHA",
        "sni": "zhiwei.li",
        "alpn": [
            "h2",
            "http/1.1"
        ],
        "reuse_session": true,
        "session_ticket": false,
        "curves": ""
    },
              

其他保持不变

ssl为了简单，也可以不验证


           "ssl": {
               "cipher": "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256",
               "sni": "zhiwei.li"
               "alpn": [
                   "http/1.1"
               ],
               "reuse_session": true,
               "session_ticket": false,
               "curves": ""
           },

但是 sni 不能为空，或者不设置



## 其他配置

参考 https://trojan-gfw.github.io/trojan/config


## 证书的获取

上面服务器用到的证书，可以 [用acme.sh来获取Let's Encrypt免费SSL证书](acme.sh来获取ssl证书.html)


## nginx原有配置改造

/etc/nginx/sites-enable/zhiwei

        listen 80;
改成
        listen 127.0.0.1:80 default_server;


然后添加一个新站点

       server {
               listen 80;
               listen [::]:80;
               server_name _;
               return 301 https://$host$request_uri;
       }


如果原站点为ssl站

        listen   1.1.1.1:443 ssl;
        listen [2605:1:2:3::9]:443 ssl;

        ssl_certificate /home/zhiwei/ssl/zhiwei.li.cer;
        ssl_certificate_key /home/zhiwei/ssl/zhiwei.li.key;

要改为 80 站点
       


## 测试并启动trojan

       trojan -c /etc/trojan/config.json -t
       systemctl restart trojan






       

    

