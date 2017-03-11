---
title: 四大组件之Service
date: 2016-10-16 20:32:13
tags: Android
---
Service是运行在主线程,在后台运行的一个组件.
这里的后台指的是它的运行是完全不依赖UI的,即即使Activity销毁了,只要进程还在Service就在.
处理耗时操作需开子线程,否则会ANR.
需要在AndroidManifest.xml中注册.
```
<service
    android:name=".MyService"
    android:exported="true" //是否需要对其他App开放,有intent-filter时默认值就是true
    android:process=":MyProcess" > //指明此进程名称,可以单独运行在新的进程中
    <intent-filter>
        <action android:name="com.example.androidtest.myservice" />
    </intent-filter>
</service>
```
<!--more-->
1.Service类型
1)普通Service
运行在后台,优先级比较低.

2)前台Service
有一个正在运行的图标在系统的状态栏显示,类似于通知栏.
```
public class MyService extends Service {  
    public static final String TAG = "MyService";  
    private MyBinder mBinder = new MyBinder();  
    @Override  
    public void onCreate() {  
        super.onCreate();  
        Notification notification = new Notification(R.drawable.ic_launcher,  
                "有通知到来", System.currentTimeMillis());  
        Intent notificationIntent = new Intent(this, MainActivity.class);  
        PendingIntent pendingIntent = PendingIntent.getActivity(this, 0,  
                notificationIntent, 0);  
        notification.setLatestEventInfo(this, "这是通知的标题", "这是通知的内容",  
                pendingIntent);  
        startForeground(1, notification);  
        Log.d(TAG, "onCreate() executed");  
    }  
    .........  
}  
```
3)IntentService
Service的一个子类,实现onHandlerIntent()方法,用于处理长期任务,并且是在新的线程中,
处理完后会自动关闭.

2.Service的生命周期
两种启动方式
1)startService (stopService结束)
onCreate -> onStartCommand -> onDestory

2)bindService (unbindService结束)
onCreate -> onBind -> onUnBind -> onDestory
