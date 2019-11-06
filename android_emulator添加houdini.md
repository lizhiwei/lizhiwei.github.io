<!--
author: lizhiwei
head:
date: 2019-10-22
title: Android Emulator添加houdini转换器
tags: android
images:
category: android
status: publish
summary: android emulator
-->


## 下载houdini

houdini        放到　/system/bin/ 目录

libhoudini.so 　放到　/system/lib 目录下

其他目录和文件　整体　放到　/system/lib/arm 目录下


##　修改属性

/system/etc/prop.default

    ro.dalvik.vm.native.bridge=libhoudini.so


/system/build.prop

    ro.enable.native.bridge.exec=1
    ...
    ro.product.cpu.abilist=x86_64,x86,armeabi-v7a,armeabi
    ro.product.cpu.abilist32=x86,armeabi-v7a,armeabi


## 对于exe文件，还需要

    mount -t binfmt_misc binfmt_misc /proc/sys/fs/binfmt_misc

    echo ':arm_exe:M::\\x7f\\x45\\x4c\\x46\\x01\\x01\\x01\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x02\\x00\\x28::/system/bin/houdini:P' > /proc/sys/fs/binfmt_misc/register
    echo ':arm_dyn:M::\\x7f\\x45\\x4c\\x46\\x01\\x01\\x01\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x03\\x00\\x28::/system/bin/houdini:P' > /proc/sys/fs/binfmt_misc/register


## 有的人觉得放文件靠谱点

    cp /system/etc/binfmt_misc/arm_exe /proc/sys/fs/binfmt_misc/register
    cp /system/etc/binfmt_misc/arm_dyn /proc/sys/fs/binfmt_misc/register


## Android Pie

platform/system/core/rootdir/etc/ld.config.txt 中添加

    namespace.default.permitted.paths += /system/${LIB}/arm
    namespace.default.permitted.paths += /system/${LIB}/arm/nb

在Android 10之后，这个文件用　　https://android.googlesource.com/platform/system/linkerconfig　编译出的　可执行文件　来动态生成




