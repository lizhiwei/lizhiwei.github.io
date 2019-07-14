<!--
author: lizhiwei
head: 
date: 2019-05-03
title: 编译boringtun
tags: IPv6
images: 
category: rust
status: publish
summary: 准备rust编译环境，编译boringtun
-->


## 准备Linux编译环境

     apt install build-essential

## 准备rust编译环境

    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh


It will add the cargo, rustc, rustup and other commands to Cargo's bin 
directory, located at:

  ~/.cargo/bin

This path will then be added to your PATH environment variable by modifying the
profile file located at:

  /~.profile

Current installation options:

   default host triple: x86_64-unknown-linux-gnu
     default toolchain: stable
  modify PATH variable: yes



## 准备borinttun源代码

    git clone  https://github.com/cloudflare/boringtun.git


## 修改boringtun代码

因为OpenVZ上不能用multi_queue，所以默认选择关闭


    diff --git a/src/main.rs b/src/main.rs
    index eede8e3..5779a3c 100644
    --- a/src/main.rs
    +++ b/src/main.rs
    @@ -132,7 +132,7 @@ fn main() {
             log_level,
             use_connected_socket: !matches.is_present("disable-connected-udp"),
             #[cfg(target_os = "linux")]
    -        use_multi_queue: !matches.is_present("disable-multi-queue"),
    +        use_multi_queue: false,  // !matches.is_present("disable-multi-queue"),
         };
    
         let mut device_handle = match DeviceHandle::new(&tun_name, config) {
    @@ -145,7 +145,7 @@ fn main() {
             }
         };
    
    -    if !matches.is_present("disable-drop-privileges") {
    +    if matches.is_present("disable-drop-privileges") {
             if let Err(e) = drop_privileges() {
                 eprintln!("Failed to drop privileges: {:?}", e);
                 sock1.send(&[0]).unwrap();

## 编译boringtun

    source $HOME/.cargo/env
    cargo build --bin boringtun --release --target x86_64-unknown-linux-gnu

 在  target/x86_64-unknown-linux-gnu/release 目录下可得到一个 boringtun的 可执行文件


## 测试

  boringtun -f wg0

如果能创建一个 wg0 的网口，就说明可以了


