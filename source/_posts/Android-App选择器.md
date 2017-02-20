---
title: Android App选择器
date: 2017-02-20 20:00:10
tags: Android
---
当手机同时装有多个同样功能的软件时,如何让系统弹出App选择界面.

例如 当用户选择打开WiFi软件时.

1.在AndroidManifest.xml中添加如下代码
```
<intent-filter>
  <action android:name="android.settings.WIFI_SETTINGS" />
  <action android:name="android.net.wifi.PICK_WIFI_NETWORK" />
  <category android:name="android.intent.category.DEFAULT" />
</intent-filter>
```
2.Java代码中调用
```
Intent intent = new Intent();
intent.setAction("android.settings.WIFI_SETTINGS");
intent.addCategory("android.intent.category.DEFAULT");
//弹出设置默认APP选择器
intent.setComponent(new ComponentName("android","com.android.internal.app.ResolverActivity"));
startActivity(intent);
```
3.判断是否已经设置了默认软件
```
/**
* 是否已经设置了默认App
*
* @param context
* @return
*/
public static boolean isSetDefaultApp(Context context) {
  Intent intent = new Intent();
  intent.setAction("android.settings.WIFI_SETTINGS");
  intent.addCategory("android.intent.category.DEFAULT");
  return hasPreferredApplication(context, intent);
}

/**
* info.activityInfo.packageName返回"android"表示没有设置默认软件
* @param context
* @param intent
* @return
*/
private static boolean hasPreferredApplication(final Context context, final Intent intent) {
  PackageManager pm = context.getPackageManager();
  ResolveInfo info = pm.resolveActivity(intent, PackageManager.MATCH_DEFAULT_ONLY);
  return !"android".equals(info.activityInfo.packageName);
}
```
