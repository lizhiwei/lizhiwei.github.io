<!--
author: lizhiwei
head: 
date: 2019-09-05
title: mitmproxy安装证书到android设备
tags: https,sniffer,mitm
images: 
category: network
status: publish
summary: 介绍mitmproxy证书的安装
-->


## 证书的位置和格式

运行mitmproxy后，会自动在　~/.mitmproxy　目录下生成证书

    mitmproxy-ca.pem　　证书和私钥
    mitmproxy-ca-cert.pem　　PEM格式的证书
    mitmproxy-ca-cert.cer　　跟PEM格式的证书完全一样，只是扩展名不同，某些android设备需要.cer后缀的文件

## Android上的证书

### Android 4.0之前
　/system/etc/security/cacert.bks 文件　含有所有系统级的CA证书
　需要将自己的证书　添加到　cacert.bks里面

### 在Android 4.0 和 ６.0 之间

/system/etc/security 目录下的所有证书文件都会被信任

但是，用户可以添加自己的用户证书到　/data/misc/keychain/certs-added 目录

### 在Android 7.0之后

默认情况下，不再信任用户证书．但是应用自己可以在AndroidManifeset.xml里配置可以信息用户证书








### 如何制作一个Android兼容的证书

#### 获得mitmproxy证书的hash值

    openssl x509 -inform PEM -subject_hash_old -in mitmproxy-ca-cert.pem | head -1

会显示　hash值为　c8750f0d

注意，要用　-subject_hash_old　选项，而不是　-subject_hash
这样才会得到　openssl 0.9 兼容的hash值

#### android证书文件名　以.0结尾

    cat mitmproxy-ca-cert.pem > c8750f0d.0
    openssl x509 -inform PEM -text -in mitmproxy-ca-cert.pem -noout >>  c8750f0d.0

#### 将证书放入　/system/etc/security/cacerts/


    adb shell
    su 
    mount -o remount,rw /system
    setenforce 0
    mv c8750f0d.0 /system/etc/security/cacerts/ 
    chmod 644 /system/etc/security/cacerts/c8750f0d.0
    setenforce 1
    mount -o remount,ro /system

#### 如果　system 分区只读，　可以考虑用magisk模块



https://github.com/Magisk-Modules-Repo/movecert

或者

https://github.com/NVISO-BE/MagiskTrustUserCerts


#### 证书安装好后，可以在　设置 里搜索　"信任的凭据"　里查看到


![system cert](/blog/img/201909/SysCert.png "红米6A导入mitmproxy证书后")

















