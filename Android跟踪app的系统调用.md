<!--
author: lizhiwei
head: 
date: 2019-10-19
title: Android跟踪app的系统调用
tags: Android
images: 
category: Android
status: publish
summary: 修改模拟器中修改网卡名称
-->


## trace.sh

    while true; do
      while ! ps  | grep -q -i $1; do :; done;
      ps | grep -i $1 | while read a b c; do
       strace -e open -f -e trace=file,ptrace -p $b 2>&1;
      done;
    done

##  使用

    sh /sdcard/strace.sh com.tencent.mm
　　

## 分析

1. 找到目标进程的pid
2. 执行

    strace -e open -f -e trace=file,ptrace -p  $pid

命令选项

    -f follow forks
    
    -p pid trace process with process id PID, may be repeated
    
    -e expr a qualifying expression: option=[!]all or option=[!]val1[,val2]…
    　　　　　　　 options: trace, abbrev, verbose, raw, signal, read, write, fault


所以，　-e open　　就是　　过滤　open 操作

