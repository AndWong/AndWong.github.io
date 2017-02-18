---
title: 四大组件之BroadcastReceiver
date: 2016-10-18 20:51:00
tags: Android
---
>BroadcastReceiver的作用有些类似全局的事件监听器，当整个手机的某一个APP或者手机自身触发事件，BroadcastReceiver接受并且处理，而且这种信号程序自身也可以创建。

一.启动方式
1.静态注册
在AndroidManifest.xml中注册,需要填写action即广播地址.
是常驻型,即使应用退出依旧可以接收到广播.
```
<receiver android:name=".MyBroadcastReceiver">  
    <intent-filter>  
        <action android:name="android.intent.action.MyBroadcastReceiver"></action>  
    </intent-filter>  
</receiver>
```
2.动态注册
在代码中注册,非"常驻型",不用时需调用unregisterReceiver否则会报错.
```
注册:
MyBroadcastReceiver receiver=new MyBroadcastReceiver();
IntentFilter filter=new IntentFilter();  
filter.addAction("android.intent.action.MyBroadcastReceiver");  
registerReceiver(receiver, filter);  

解除注册:
unregisterReceiver(receiver);
```

二.广播类型
1.无序广播
各个广播直接的接收是异步的,彼此无影响
sendBroadcast() 发送广播
abortBroadcast() 终止广播

2.有序广播
按优先级获取, priority(-1000到1000),
sendOrderedBroadcast() 发送广播
abortBroadcast() 终止广播
1)setResultExtras 传递给下一个
2)getResultExtras 获取上一个
