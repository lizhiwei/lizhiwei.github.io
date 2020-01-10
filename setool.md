<!--
author: lizhiwei
head: 
date: 2019-11-21
title: Android Sepolicy与setool
tags: Android
images: 
category: Android
status: publish
summary: selinux
-->


## setools for android

https://github.com/xmikos/setools-android.git



##　在高版本Android上 /sepolicy 文件不存在



       diff --git a/jni/libqpol/util.c b/jni/libqpol/util.c
       index 7e2dc2b..101ef95 100644
       --- a/jni/libqpol/util.c
       +++ b/jni/libqpol/util.c
       @@ -81,7 +81,7 @@ static int get_binpol_version(const char *policy_fname)
        
        static int search_policy_binary_file(char **path)
        {
       -       const char *binary_path = "/sepolicy";
       +       const char *binary_path = "/sys/fs/selinux/policy";
        
               int expected_version = -1, latest_version = -1;
        #ifdef LIBSELINUX


改成 /sys/fs/selinux/policy 就可以了


### 搜索某个特定class相关的规则


       sesearch -A  -c netlink_route_socket      



### netlink_route_socket

继承自common socket

  https://selinuxproject.org/page/ObjectClassesPerms#netlink_route_socket


