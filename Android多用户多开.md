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

hook住  canAddMoreManagedProfiles  和  getNextAvailableId


createProfileForUser 传递 disallowedPackages




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
  
  


然后
startUserInBackground
installBasicApp




