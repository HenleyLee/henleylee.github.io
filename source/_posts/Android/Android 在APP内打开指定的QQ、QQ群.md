---
title: Android 在APP内打开指定的QQ、QQ群
date: 2018-07-21 09:20:35
categories: Android
tags:
  - Android
---

## 1.判断是否安装 QQ 客户端 ##
```java
    /**
     * QQ包名
     */
    public static final String PACKAGENAME_QQ = "com.tencent.mobileqq";

    /**
     * 判断应用是否已安装
     */
    public static boolean checkApkInstalled(Context context, String packageName) {
        if (TextUtils.isEmpty(packageName)) {
            return false;
        }
        try {
            ApplicationInfo info = context.getPackageManager().getApplicationInfo(packageName, 0);
            return info != null;
        } catch (PackageManager.NameNotFoundException e) {
            return false;
        }
    }
 ```
 
## 2.打开指定的 QQ 聊天页面 ##
```java
    /**
     * 打开指定的QQ聊天页面
     *
     * @param context 上下文
     * @param QQ      QQ号码
     */
    public static boolean openQQChat(Context context, String QQ) {
        try {
            String url = "mqqwpa://im/chat?chat_type=wpa&uin=" + QQ;
            Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
            context.startActivity(intent);
            return true;
        } catch (ActivityNotFoundException e) {
            return false;
        }
    }
```
> 注意：该方法指定的 QQ 号码如果没有添加为好友，此 QQ 号码需要在 [QQ 推广官网](http://wp.qq.com/set.html) 开通 QQ 推广功能，否则向此 QQ 号发送临时消息会发送失败。

## 3.打开指定的 QQ 群 ##
### 方法一： ###
```java
    /**
     * 打开指定的QQ群聊天页面
     *
     * @param context 上下文
     * @param group   QQ群号码
     */
    public static boolean openQQGroup(Context context, String group) {
        try {
            String url = "mqqwpa://im/chat?chat_type=group&uin=" + group;
            Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
            context.startActivity(intent);
            return true;
        } catch (ActivityNotFoundException e) {
            return false;
        }
    }
```
> 注意：该方法只能打开已经加入的 QQ 群。

### 方法二： ###
```java
    /**
     * 打开指定的QQ群聊天页面
     *
     * @param context 上下文
     * @param key     由QQ官网生成的Key
     */
    public static boolean joinQQGroup(Context context, String key) {
        try {
            String url = "mqqopensdkapi://bizAgent/qm/qr?url=http%3A%2F%2Fqm.qq.com%2Fcgi-bin%2Fqm%2Fqr%3Ffrom%3Dapp%26p%3Dandroid%26k%3D" + key;
            Intent intent = new Intent();
            intent.setData(Uri.parse(url));
            // 此Flag可根据具体产品需要自定义，如设置，则在加群界面按返回，返回手Q主界面，不设置，按返回会返回到呼起产品界面
            // intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            context.startActivity(intent);
            return true;
        } catch (ActivityNotFoundException e) {
            return false;
        }
    }
```
> 注意：打开指定的 QQ 群需要的 `Key` 可以在 [QQ群官网](https://qun.qq.com/join.html) 选择需要的 QQ 群生成对应的 Key。
