---
title: SecurityException与android.permission.xxxx
date: 2016-08-30 10:33:27
tags: problems
---
~~~java
java.lang.SecurityException
getDeviceId: Neither user 10174 nor current process has android.permission.READ_PHONE_STATE.

~~~
这个问题是由于android6.0的动态权限引起的
首先看下google怎么说的
Beginning in Android 6.0 (API level 23), users grant permissions to apps while the app is running, not when they install the app. This approach streamlines the app install process, since the user does not need to grant permissions when they install or update the app. It also gives the user more control over the app's functionality; for example, a user could choose to give a camera app access to the camera but not to the device location. The user can revoke the permissions at any time, by going to the app's Settings screen.

***
从6.0起，权限分成两类，一个是普通权限，一个是危险权限
- 普通权限就是手机本身的权限，跟以前一样在AndroidManifest.xml里面申请就可以了
- 危险权限就是需要获取用户信息的一些权限，比如

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2524531-ade5fbdd6a047148.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在activity里面，加入这俩方法
~~~java
 public void getPermission(){
                if (Build.VERSION.SDK_INT >= 23) {
            int checkCallPhonePermission = ContextCompat.checkSelfPermission(mContext,Manifest.permission.READ_PHONE_STATE);
            if(checkCallPhonePermission != PackageManager.PERMISSION_GRANTED){
                ActivityCompat.requestPermissions(mContext,new String[]{Manifest.permission.READ_PHONE_STATE},REQUEST_CODE_READ_PHONE_STATE);
                return;
            }else{
                //原本需要做的事情
            }
        } else {
            //原本需要做的事情
        }
    }
~~~
~~~java
@Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        switch (requestCode) {
            case REQUEST_CODE_READ_PHONE_STATE:
                if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    // Permission Granted
                    //原本需要做的事情
                    
                } else {
                    // Permission Denied
                    Toast.makeText(this, "READ_PHONE_STATE PERMISSION Denied", Toast.LENGTH_SHORT)
                            .show();
                }
                break;
            default:
                super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        }
    }
~~~