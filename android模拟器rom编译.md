<!--
author: lizhiwei
head: 
date: 2019-11-06
title: android模拟器rom编译
tags: android
images: 
category: android
status: publish
summary: 给android模拟器器编译ROM
-->



##　在Debian 上准备一些依赖的包

    apt install git  build-essential g++-multilib  gcc-multilib autoconf automake libtool flex bison gdb   libncurses5  zip  libxml2-utils curl

在Debian 10(buster) Debian 11 (bullseys)上都是这些
　　注意是　libncurses5 而不是系统中已经安装好的　libncurses6

    　

## 准备repo脚本

    mkdir ~/bin
    PATH=~/bin:$PATH
    curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
    chmod a+x ~/bin/repo

## 下载aosp源代码

    wget -c https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar
    tar xf aosp-latest.tar
    cd AOSP
    repo sync

## 切换版本（可选）
   如果想编译其他分支的源代码， 在执行 repo sync 命令前需要进行切换


       repo init -b android-9.0.0_r50


## 清理

    make clobber

## 准备编译环境，配置编译类型

    . build/envsetup.sh
    lunch sdk_phone_x86_64-userdebug

##  编译

    m

## 编译好的成果在

    out/target/product/generic_x86_64

