<!--
author: lizhiwei
head: 
date: 2019-12-31
title: 用Frida分析Android多用户管理
tags: Android,Frida
images: 
category: Android
status: publish
summary: Android多用户管理
-->


## 分析 getUsers

user.js

       function StackTraceElementToString(stack){
           var at = ""
           for(var i = 0; i < stack.length; ++i){
               at += stack[i].toString() + "\n"
           }
           return at
       }


       setImmediate(function() { //prevent timeout
           Java.perform(function(){
               console.log("start hook....");

               var threadef = Java.use('java.lang.Thread');
               var threadinstance = threadef.$new();

               var UserManagerService = Java.use("com.android.server.pm.UserManagerService");
               console.log("UserManagerService Found."); 

               var getUsers = UserManagerService.getUsers.overload("boolean");
               getUsers.implementation = function (excludeDying) {
                   console.log("getUsers");
                   var stack = threadinstance.currentThread().getStackTrace();
                   var full_call_stack = StackTraceElementToString(stack);
                   console.log("\n\t         Full call stack:" + full_call_stack);
                   return this.getUsers(excludeDying); 
               };
       
               console.log("hook ok");
           });
       });

user.py

       #!/usr/bin/python3.7
       
       import sys
       sys.path.insert(1, '/home/zhiwei/frida/lib/python3.7')
       import frida
       
       def on_message(message, data):
           if message['type'] == 'send':
               print("[*] {0}".format(message['payload']))
           else:
               print(message)
       
       
       process = frida.get_device(id='a1b2c3d4').attach('system_server')
       script = process.create_script(open('user.js').read())
       script.on('message', on_message)
       
       print(' Running Frida ...')
       script.load()
       sys.stdin.read()   

在adb shell 里运行 pm list users 会看到call statck

       com.android.server.pm.UserManagerService.getUsers(Native Method)
       com.android.server.pm.UserManagerService.runList(UserManagerService.java:3672)
       com.android.server.pm.UserManagerService.onShellCommand(UserManagerService.java:3662)
       com.android.server.pm.UserManagerService$Shell.onCommand(UserManagerService.java:4138)
       android.os.ShellCommand.exec(ShellCommand.java:103)
       com.android.server.pm.UserManagerService.onShellCommand(UserManagerService.java:3650)
       android.os.Binder.shellCommand(Binder.java:634)
       com.android.server.pm.PackageManagerShellCommand.runList(PackageManagerShellCommand.java:406)
       com.android.server.pm.PackageManagerShellCommand.onCommand(PackageManagerShellCommand.java:148)


### 在1加上实现多开(5开以上)


       setImmediate(function() { //prevent timeout
           Java.perform(function(){
               console.log("start hook....");
       
               var threadef = Java.use('java.lang.Thread');
               var threadinstance = threadef.$new();
       
               var UserManagerService = Java.use("com.android.server.pm.UserManagerService");
               console.log("UserManagerService Found.");   
               
               var UserInfo = Java.use("android.content.pm.UserInfo");
               console.log(UserInfo.FLAG_MANAGED_PROFILE.value);
       
               var ums = UserManagerService.getInstance();
               ums.mNextSerialNumber.value = 990;
               
               UserManagerService.getNextAvailableId.overload().implementation = function () {
                   var aid = this.getNextAvailableId(); 
                   console.log("new id: " + aid);
                   return 991;
               };
               
       
               UserManagerService.canAddMoreManagedProfiles.overload('int', 'boolean').implementation = function(userId, allowedToRemoveOne) {
                   return true;
               }
       
               UserManagerService.createUserInternalUnchecked.implementation = function(name, flags, parentId, disallowedPackages) {
                   console.log("disallowedPackages: " + disallowedPackages);
                   disallowedPackages = ["android.autoinstalls.config.oneplus.oneplus6t", "android.ext.services", "android.ext.shared", "android.telephony.overlay.cmcc",
                       "cn.oneplus.nvbackup", "cn.oneplus.photos", "cn.oneplus.wallet", "com.amap.android.ams", "com.android.apps.tag", "com.android.backupconfirm",
                       "com.android.bips", "com.android.bluetooth", "com.android.bluetoothmidiservice", "com.android.browser", "com.android.calllogbackup", "com.android.captiveportallogin", 
                       "com.android.carrierconfig", "com.android.carrierdefaultapp", "com.android.cellbroadcastreceiver", "com.android.certinstaller", "com.android.companiondevicemanager",
                       "com.android.contacts", "com.android.cts.ctsshim", "com.android.cts.priv.ctsshim", "com.android.defcontainer", "com.android.dialer", "com.android.dreams.basic", 
                       "com.android.egg", "com.android.emergency", "com.android.exchange", "com.android.htmlviewer", "com.android.inputdevices", "com.android.inputmethod.latin", 
                       "com.android.internal.display.cutout.emulation.tall", "com.android.location.fused", "com.android.managedprovisioning", "com.android.mtp",
                       "com.android.networksettings.overlay.ct", "com.android.nfc", "com.android.onetimeinitializer", "com.android.pacprocessor", "com.android.printservice.recommendation", 
                       "com.android.protips", "com.android.providers.blockednumber", "com.android.providers.downloads.ui", "com.android.providers.userdictionary",
                       "com.android.proxyhandler", "com.android.se", "com.android.settings.intelligence", "com.android.sharedstoragebackup", "com.android.shell",
                       "com.android.smspush", "com.android.statementservice", "com.android.stk", "com.android.storagemanager", "com.android.systemui", "com.android.traceur",
                       "com.android.wallpaper.livepicker", "com.android.wallpaperbackup", "com.android.wallpapercropper", "com.android.wallpaperpicker", "com.baidu.input_yijia", 
                       "com.coloros.mcs", "com.dsi.ant.server", "com.example.rftuner", "com.example.wifirftest", "com.fingerprints.fingerprintsensortest", "com.google.android.partnersetup", 
                       "com.google.android.syncadapters.contacts", "com.nearme.atlas", "com.nearme.browser", "com.oem.logkitsdservice", "com.oem.nfc", "com.oem.oemlogkit",
                       "com.oneplus.account", "com.oneplus.aod", "com.oneplus.applocker", "com.oneplus.backuprestore", "com.oneplus.backuprestore.remoteservice", "com.oneplus.brickmode",
                       "com.oneplus.factorymode.specialtest", "com.oneplus.findmyphoneutils", "com.oneplus.gallery", "com.oneplus.iconpack.circle", "com.oneplus.iconpack.h2default", 
                       "com.oneplus.filemanager", "com.oneplus.findmyphone", "com.oneplus.iconpack.h2folio", "com.oneplus.soundrecorder", "com.oppo.market", "com.qti.confuridialer",
                       "com.qti.dpmserviceapp", "com.qti.qualcomm.datastatusnotification", "com.qti.qualcomm.deviceinfo", "com.qualcomm.embms", "com.qualcomm.location", "com.qualcomm.qcrilmsgtunnel",
                       "com.qualcomm.qti.callfeaturessetting", "com.qualcomm.qti.dynamicddsservice", "com.qualcomm.qti.ims", "com.qualcomm.qti.lpa", "com.qualcomm.qti.networksetting",
                       "com.qualcomm.qti.optinoverlay", "com.qualcomm.qti.poweroffalarm", "com.qualcomm.qti.qtisystemservice", "com.qualcomm.qti.seccamservice", "com.qualcomm.qti.simsettings",
                       "com.qualcomm.qti.smcinvokepkgmgr", "com.qualcomm.qti.telephonyservice", "com.qualcomm.qti.uceShimService", "com.qualcomm.qti.uim", "com.qualcomm.timeservice", "com.qualcomm.uimremoteclient",
                       "com.qualcomm.wfd.service", "com.quicinc.cne.CNEService", "com.redteamobile.oneplus.roaming", "com.redteamobile.virtual.softsim", "com.tencent.soter.soterserver",
                       "com.wapi.wapicertmanage", "net.oneplus.commonlogtool", "net.oneplus.launcher", "net.oneplus.odm", "net.oneplus.odm.provider", "net.oneplus.provider.appcategoryprovider",
                       "net.oneplus.wallpaperresources", "net.oneplus.weather", "net.oneplus.widget", "org.codeaurora.bluetooth", "org.codeaurora.ims", "se.dirac.acs"
                       ]; 
                       
                   return this.createUserInternalUnchecked(name, flags, parentId, disallowedPackages);
               }
       
       
               console.log("hook ok");
           });
       });


