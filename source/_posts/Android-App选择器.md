---
title: Android App选择器
date: 2017-02-20 20:00:10
tags: Android
---
当手机同时装有多个同样功能的软件时,如何让系统弹出App选择界面.

例如 当用户选择打开WiFi软件时.

一.先来介绍第一种方式:

1.在AndroidManifest.xml中为目标Activity添加如下代码
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
通过上述方式,虽然可以成功启动APP选择器,
但是会有一个问题,
如果用户已经设置其他app为默认应用,那么此次的设置就会无效.

接下来我将介绍的第二种方式可以解决上述的问题.
通过在AndroidManifest.xml中为多个activity配置intent-filter,
在调用startActivity时系统不知道要匹配哪个intent-filter便会弹出选择器.

代码如下:
1.这个Activity什么都不用做连布局都不需要,只需要在xml中配置intent-filter
以及enabled=false和exported=false
```
public class DefaultSetting extends Activity{

}
```
AndroidManifest.xml中配置
```
<!--　enabled和exported很重要　-->
<activity
    android:name="com.wifibanlv.wifipartner.usu.activity.DefaultSetting"
    android:enabled="false"
    android:exported="false">
    <intent-filter>
      <action android:name="android.settings.WIFI_SETTINGS" />
      <action android:name="android.net.wifi.PICK_WIFI_NETWORK" />
      <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</activity>
```

2.这个Activity为调用startActivity时匹配的目标Activity,
也可将它作为一个中转界面,监听设置是否成功.
同样不需要布局
```
public class DefaultSettingTransfer extends Activity{
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ToastUtil.showMessage("设置成功~");
        startActivity(new Intent(this, MainActivity.class));
        finish();
    }
}
```
在AndroidManifest.xml中配置
```
<!-- 设置默认wifi工具中转界面 -->
<activity
  android:name="com.wifibanlv.wifipartner.activity.DefaultSettingTransfer"
  android:icon="@drawable/ic_launcher"
  android:label="@string/app_name"
  android:screenOrientation="portrait"
  android:theme="@style/AppTheme">
  <intent-filter>
    <action android:name="android.settings.WIFI_SETTINGS" />
    <action android:name="android.net.wifi.PICK_WIFI_NETWORK" />
    <category android:name="android.intent.category.DEFAULT" />
  </intent-filter>
</activity>
```
3.在Java代码中调用启动App选择器的方法,如下:
```
PackageManager packageManager = context.getPackageManager();
//设置activity的enable=true,会出现多个匹配项系统就会弹出选择器
ComponentName componentName = new ComponentName(App.getInstance().getPackageName(), "com.wifibanlv.wifipartner.usu.activity.DefaultSetting");
packageManager.setComponentEnabledSetting(componentName, PackageManager.COMPONENT_ENABLED_STATE_ENABLED, PackageManager.DONT_KILL_APP);
Intent intent = new Intent();
intent.setAction("android.settings.WIFI_SETTINGS");
intent.addCategory("android.intent.category.DEFAULT");
startActivity(intent);
//设置activity的enable=false;系统会选择enable=true的activity作为目标activity
packageManager.setComponentEnabledSetting(componentName, PackageManager.COMPONENT_ENABLED_STATE_DEFAULT, PackageManager.DONT_KILL_APP);
```
这个设置之后,即使用户已经设置其他app为默认app,我们也同样能重新设置.

二.判断是否已经设置了默认app
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
* 若info.activityInfo.packageName返回"android"表示未设置默认应用
* 返回哪个应用的包名表示设置哪个应用为默认应用
* @param context
* @param intent
* @return
*/
private static boolean hasPreferredApplication(final Context context, final Intent intent) {
  PackageManager pm = context.getPackageManager();
  ResolveInfo info = pm.resolveActivity(intent, PackageManager.MATCH_DEFAULT_ONLY);
  return App.getInstance().getPackageName().equals(info.activityInfo.packageName);
}
```
