<!--
author: lizhiwei
head: 
date: 2019-10-19
title: frida如何从js传递byte array到java或者native
tags: frida
images: 
category: Java
status: publish
summary: frida如何从js传递byte array到java
-->


## frida作者oleavr 提供的方法

    var buffer = Java.array('byte', [ 13, 37, 42 ]);

##  LTE auth 的一个例子


    Java.perform(function () {

        function hexToBytes(hex) {
            for (var bytes = [], c = 0; c < hex.length; c += 2)
            bytes.push(parseInt(hex.substr(c, 2), 16));
            return bytes;
        }
    
        function bytesToHex(bytes) {
            for (var hex = [], i = 0; i < bytes.length; i++) { hex.push(((bytes[i] >>> 4) & 0xF).toString(16).toUpperCase());
                hex.push((bytes[i] & 0xF).toString(16).toUpperCase());
                hex.push(" ");
            }
            return hex.join("");
        }

        send(Java.androidVersion);
        var Cmd = Java.use("li.zhiwei.Cmd");
    
        var hex_cmd = "008800812210AD616CBBF0ACF36E9D87490FC78084331050C2E3B3E0B1800047687BF6D0EFF951";
        var pdu_cmd = Java.array('byte', hexToBytes(hex_cmd));
        var resp_bytes = Cmd.process(pdu_cmd);
        var resp_hex = bytesToHex(resp_bytes)
    
        send(resp_hex);
    });



## 到native

    var st = Memory.alloc(39);
    Memory.writeByteArray(st, [0x00, 0x88, 0x00, 0x81, 0x22, 0x10, 0xAD, 0x61, 0x6C, 0xBB, 0xF0, 0xAC, 0xF3, 0x6E, 0x9D, 0x87, 0x49, 0x0F, 0xC7, 0x80, 0x84, 0x33, 0x10, 0x50, 0xC2, 0xE3, 0xB3, 0xE0, 0xB1, 0x80,  0x00, 0x47, 0x68, 0x7B, 0xF6, 0xD0, 0xEF, 0xF9, 0x51]);





