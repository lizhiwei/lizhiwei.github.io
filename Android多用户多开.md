<!--
author: lizhiwei
head: 
date: 2019-12-31
title: Android用多用户实现应用多开
tags: Android
images: 
category: Android
status: publish
summary: Android应用多开
-->


## AOSP中的修改


### framework/base/core/res/res/values/config.xml

       config_multiuserMaximumUsers  支持的最多用户数
       config_multiuserMaxRunningUsers 允许同时运行的用户数
       config_enableMultiUserUI
       

### framework/base/services/core/java/com/android/server/pm/UserManagerService.java


private UserInfo createUserInternalUnchecked(String name, int flags, int parentId, String[] disallowedPackages) {
屏蔽掉

         if (isManagedProfile && !canAddMoreManagedProfiles(parentId, false)) {
              Log.e(LOG_TAG, "Cannot add more managed profiles for user " + parentId);
              return null;
         }

在Oneplus里是家了一个特殊的flag, 判断这个flag是oneplus(0x04000000), 就不执行上面的return null



com/android/server/pm/PackageManagerService.java:    void createNewUser(int userId, String[] disallowedPackages) {




### 在app里实现

    public boolean createUserProfile(){
        UserHandle userHandle = android.os.Process.myUserHandle();
        int userHandleIdentifier = userHandle.getIdentifier();
        mManagedProfileUserInfo = userManager.createProfileForUserEvenWhenDisallowed(createUserName(), UserInfo.FLAG_MANAGED_PROFILE, userHandleIdentifier, disallowedPackages);
       
        if(mManagedProfileUserInfo != null){
            Log.i(TAG,"createUserProfile 成功!!" );
            startNewUser();
            return true;
        }
        return false;
    }



ALLOWED_FLAGS_FOR_CREATE_USERS_PERMISSION  =
            UserInfo.FLAG_MANAGED_PROFILE    ( 0x00000020 )
            | UserInfo.FLAG_EPHEMERAL         ( 0x00000100 ) 转入后台，就会被删除
            | UserInfo.FLAG_RESTRICTED       （ 0x00000008）权限限制，不能安装app或者管理wif
            | UserInfo.FLAG_GUEST            ( 0x00000004 )
            | UserInfo.FLAG_DEMO;              (0x00000200)


创建用户和Profile的区别：
用户的父亲，是 UserHandle.USER_NULL （-10000）， 而Profile的父亲 必须是 普通用户




### 一加的实现

com.oneplus.settings.multiapp.OPMultiAppListSettings


UserManager.setUserRestriction("no_add_user", false, UserHandle.OWNER)

UserManager.createProfileForUser('username', 0x4000060, Process.myUserHandle().getIdentifier());


            标志： 一加， 0x20(FLAG_MANAGED_PROFILE), 0x40 (FLAG_DISABLED)

然后删掉不必要的应用


在UserManagerService 中  对 createUserInternalUnchecked 进行了改造

canAddMoreManagedProfile 通过Oppo标志直接屏蔽


UserId直接改成  999
                 if ((OpFeatures.isSupport(v6_1))&&((0x04000000&xFlags))) {	
                     v0 = 999;
                 }else {	
                     v0_3 = this.getNextAvailableId();
                 }




Profile创建好之后


UserManager.setUserName(ManagedProfileOrUserInfo.id, this.getString(0x7F120C14)); //设置名称
UserManager.setUserEnabled(ManagedProfileOrUserInfo.id);  //启用

startUserInBackground(id)

安装已经存在的包
PackageManager.installExistingPackageAsUser



### AOSP中管理Profile的代码

aosp/developers/samples/android/admin/BasicManagedProfile
aosp/development/samples/browseable/BasicManagedProfile
aosp/packages/apps/ManagedProvisioning
  ... startManagedProfileOrUserProvisioning



### PackageManagerShellCommand 分析

runCreateUser()


pm create-user --profileOf  0  --managed

最终到  createUserInternalUnchecked


hook住  **canAddMoreManagedProfiles ** 和  **getNextAvailableId**


createProfileForUser 传递 disallowedPackages



       adb shell pm create-user --profileOf 0 --managed profile1
       adb shell pm create-user --profileOf 0 --managed profile2




setprop fw.max_users 10



### 联想


    private UserInfo getUsefulUser(String userName, String pkgName, boolean isFensheng) {
        UserInfo userInfo;
        StringBuilder sb = new StringBuilder();
        UserManager userManager = (UserManager) this.mContext.getSystemService("user");
        List<UserInfo> users = userManager.getUsers();
        this.mMultipleSpaceCount = 0;
        users.sort(new Comparator<UserInfo>() {
            public int compare(UserInfo userInfo, UserInfo userInfo2) {
                return userInfo.getMultipleSpaceId() - userInfo2.getMultipleSpaceId();
            }
        });
        if (users == null || users.size() <= 1) {
            userInfo = null;
        } else {
            userInfo = null;
            for (UserInfo userInfo2 : users) {
                boolean isSecuritySpaceUser = MultiSpaceConstant.isSecuritySpaceUser(userInfo2.id);
                if ((!isFensheng && isSecuritySpaceUser) || (isFensheng && userInfo2 != null && userInfo2.isMultipleSpace() && !isSecuritySpaceUser)) {
                    if (TextUtils.isEmpty(sb.toString())) {
                        sb.append(userInfo2.id);
                    } else {
                        sb.append("," + userInfo2.id);
                    }
                    if (!isSecuritySpaceUser) {
                        this.mMultipleSpaceCount++;
                    }
                    if (userInfo == null && ((!isFensheng && isSecuritySpaceUser) || !isInstalled(pkgName, userInfo2.id))) {
                        userInfo = userInfo2;
                    }
                }
            }
        }
        this.mSplitUserIsExist = true;
        if (userInfo != null) {
            return userInfo;
        }
        if (this.mMultipleSpaceCount >= 4 && !isFensheng) {    //因为isFensheng为true, 这个条件永不满足
            return userInfo;
        }
        this.mSplitUserIsExist = false;
        Settings.Secure.putString(this.mContext.getContentResolver(), KEY_MULTI_SPACE_USERS, sb.toString());
        UserInfo createUser = userManager.createUser(userName, getUserInfoFlag(isFensheng) | 16);
        Settings.Secure.putString(this.mContext.getContentResolver(), KEY_MULTI_SPACE_USERS, (String) null);
        Log.v(TAG, "Creating managed profile with name " + userName + " " + pkgName + " targetUserInfo = " + createUser);
        return createUser;
    }


 FLAG_MULTIPLE_SPACE = 0x80000800;
 FLAG_SECURITY_SPACE = 0x80001000;
 FLAG_MANAGED_PROFILE = 0x20
 FLAG_MASK_MULTIPLE_SPACE_ID = 0xFF00000
  FLAG_MASK_USER_TYPE = 0xFFFF
  FLAG_INITIALIZED = 0x10  (创建时带标志，不会初始化一些包)
  
  
联想魔改了 android.content.pm.UserInfo, 所以它用的createuser创建的分身，而不是creatUserProfile


    public boolean isSecuritySpace() {
        return (this.flags & 0x80001000) == 0x80001000;
    }

    public boolean isMultipleSpace(boolean excludeSecurity) {
        return (this.flags & 0x80000800) == 0x80000800 || (!excludeSecurity && (this.flags & 0x80001000) == 0x80001000);
    }
    
    public boolean isMultipleSpace() {
        return isMultipleSpace(false);
    }


然后
startUserInBackground
installBasicApp

#### 联想对switchUser的魔改
com.android.server.am.UserController

    public boolean switchUser(int targetUserId) {
        enforceShellRestriction("no_debugging_features", targetUserId);
        int currentUserId = getCurrentUserId();
        UserInfo targetUserInfo = getUserInfo(targetUserId);
        if (targetUserId == currentUserId) {
            Slog.i(TAG, "user #" + targetUserId + " is already the current user");
            return true;
        } else if (targetUserInfo == null) {
            Slog.w(TAG, "No user info for user #" + targetUserId);
            return false;
        } else if (!targetUserInfo.supportsSwitchTo()) {
            Slog.w(TAG, "Cannot switch to User #" + targetUserId + ": not supported");
            return false;
        } else if (targetUserInfo.isManagedProfile() || (UserHandle.supportsMultiSpace() && targetUserInfo.isMultipleSpace())) {
            Slog.w(TAG, "Cannot switch to User #" + targetUserId + ": not a full user");
            return false;
        } else {
            synchronized (this.mLock) {
                this.mTargetUserId = targetUserId;
            }
            if (this.mUserSwitchUiEnabled) {
                Pair<UserInfo, UserInfo> userNames = new Pair<>(getUserInfo(currentUserId), targetUserInfo);
                this.mUiHandler.removeMessages(1000);
                this.mUiHandler.sendMessage(this.mHandler.obtainMessage(1000, userNames));
            } else {
                this.mHandler.removeMessages(START_USER_SWITCH_FG_MSG);
                this.mHandler.sendMessage(this.mHandler.obtainMessage(START_USER_SWITCH_FG_MSG, targetUserId, 0));
            }
            return true;
        }


## 小米的修改

com.miui.xspace.XSpaceCompat

       createProfileForUser(profileName, 0x800040, Process.myUserHandle().getIdentifier());

...

也在com.android.server.pm.UserManagerService 中  对 createUserInternalUnchecked 进行了改造

if(is_xiaomi_dual != 0 && !v1.canAddMoreManagedProfiles(parentUserId, false) && (0x800000 & myflags) == 0) {
                Log.e("UserManagerService", "Cannot add more managed profiles for user " + parentUserId);
                __monitor_exit(v14);
                goto label_75;
            }


android.content.pm.UserInfo

public static final int FLAG_DISABLED = 0x40;
public static final int FLAG_AIR_SPACE = 0x400000;
public static final int FLAG_XSPACE_PROFILE = 0x800000;

public boolean isAirSpace() {
        return (this.flags & 0x400000) == 0x400000;
}

public boolean isManagedProfile() {
     return (this.flags & 0x20) == 0x20 ;
}

public boolean isEnabled() {    //用来防止切换成前台用户
     return (this.flags & 0x40) != 0x40;
}



## 跟踪不需要安装的包

com/android/server/pm/PackageManagerService.java

    /** Called by UserManagerService */
    void createNewUser(int userId, String[] disallowedPackages) {
        synchronized (mInstallLock) {
            mSettings.createNewUserLI(this, mInstaller, userId, disallowedPackages);
        }
        synchronized (mPackages) {
            scheduleWritePackageRestrictionsLocked(userId);
            scheduleWritePackageListLocked(userId);
            applyFactoryDefaultBrowserLPw(userId);
            primeDomainVerificationsLPw(userId);
        }
    }


com.android.server.pm.Settings

void createNewUserLI(@NonNull PackageManagerService service, @NonNull Installer installer,
            int userHandle, String[] disallowedPackages)


                final boolean shouldInstall = ps.isSystem() &&
                        !ArrayUtils.contains(disallowedPackages, ps.name);
                // Only system apps are initially installed.
                ps.setInstalled(shouldInstall, userHandle);
                if (!shouldInstall) {
                    writeKernelMappingLPr(ps);
                }

       只有系统应用，并且在 disallowedPackages 列表的，才可以安装


## 小米必须安装

android   （/system/framework/framework-res.apk）
com.android.providers.settings  （/system/priv-app/SettingsProvider/SettingsProvider.apk）
com.google.android.webview   （/system/app/WebViewGoogle/WebViewGoogle.apk）



## 联想必须装

com.google.android.webview
com.lenovo.lsf

## 一加必须装

android   （/system/framework/framework-res.apk）
com.android.providers.settings  （/system/priv-app/OPSettingsProvider/OPSettingsProvider.apk）
com.google.android.webview       （/system/app/GoogleWebView/GoogleWebView.apk）





