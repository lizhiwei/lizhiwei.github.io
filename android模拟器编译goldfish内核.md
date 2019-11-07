<!--
author: lizhiwei
head: 
date: 2019-11-06
title: android模拟器编译goldfish内核
tags: android
images: 
category: android
status: publish
summary: android模拟器编译goldfish内核
-->



##　用aosp里prebuilts的build-kernel脚本

    git clone git://mirrors.ustc.edu.cn/aosp/kernel/goldfish.git
    cd goldfish
    git checkout  android-goldfish-4.4-dev
    ../rom/prebuilts/qemu-kernel/build-kernel.sh  --arch=x86_64  --config=x86_64_ranchu


## ARM64版本

    cd aosp
    . build/envsetup.sh
    lunch sdk_phone_arm64-userdebug
    
    cd ../goldfish
    ../aosp/prebuilts/qemu-kernel/build-kernel.sh  --arch=arm64  --config=ranchu64

## ARM版本


    ../aosp/prebuilts/qemu-kernel/build-kernel.sh  --arch=arm  --config=ranchu


