<!--
author: lizhiwei
head: 
date: 2019-11-06
title: 编译android emulaotr
tags: android
images: 
category: android
status: publish
summary: 编译android emulaotr
-->



##　源代码在aosp中

    cd asop
    repo init -b emu-29.0-release
    repo sync
    cd external/qemu/android/


现在模拟器版本标号，跟 SDK编号同步了
SDK 29 对应的　emu-29.0


## 编译

    rebuild.sh --clean=0

## 现在改用cmake编译了

    配置程序是python脚本

    build/python/aemu/cmake.py

可以看出
--tests=0 不进行测试
--clean=0 不清除之前的编译


